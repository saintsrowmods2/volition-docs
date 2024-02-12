# Zone files

??? note "Links"
    - [Knobby | SR3 zone file format](https://www.saintsrowmods.com/forum/threads/sr3-zone-file-format.2855/)
    - [Kinzie's Toy Box | File Formats](https://github.com/saintsrowmods2/Kinzies-Toy-Box/blob/master/file_formats.md)
!!! end "Tools"
    [`quantum_sr_zone_tools`](/tools/quantum_sr_zone_tools)
___

The zone file is split up into a header(czh file) and the zone data itself (czn file). We did this to save memory since we only need the header portion for building streaming containers and at load time. We load and then dump this header data.

There is a [`v_file_header`](/SRTT/formats/common/#v_file_header) on top of the czh that describes the things that the zone references. This will be things like textures and meshes and they end up in the str2 file.

```cpp title="v_file_header"
uint16 signature - 0x3854
uint16 version - 4
uint32 reference_data_size
uint32 reference_data_start(offset)
uint32 reference_count - number of references in the file
uint8 pad[16] - zero'd before write, padding so we can add to the header
```

Reference data always follows the header. That data starts at reference_data_start offset from the top of the v_file_header and to get to the next reference(string) you just advance past the string. Double null termination on this string list, but the size is known as well.

Immediately following the v_file_header is the world zone header:
```cpp title="world_zone_header"
// Header for zones
struct world_zone_header {
    // 8 bytes 
    et_uint32   m_signature;   // 'Z3RS'
    et_uint32   m_version;     // 29

    // 4/8 bytes, 8 on PC, points back up into the v_file_header since we use that at load time to find filenames(saving some space)
    const struct v_file_header *m_v_file_header;    // Filled in at load time, 

    et_fl_vector        m_file_reference_offset;    // position offset to apply to all file refs

    wz_file_reference   *m_file_references;     // Filled in at load time
    et_uint16           m_num_file_references;  // number of file references

    uint8               m_zone_type;    // type of zone
    uint8               m_unused;       // pad previous to two bytes

    wz_interior_trigger *m_interior_triggers;       // filled in at load time
    et_uint16           m_num_interior_triggers;    // filled in at load time

    uint16              m_extra_objects;  // only in test levels(not sure for what)

    uint8               m_pad[24]; // Pad for future use, guaranteed to be zero'd
};

```

??? info "enum zone_types"
    ```cpp
    // Types of zones
    enum world_zone_types {
    WZT_UNKNOWN = 0,
    WZT_ALWAYS_LOADED,
    WZT_STREAMING,
    WZT_STREAMING_AL,
    WZT_TEST_LEVEL,
    WZT_MISSION,
    WZT_ACTIVITY,
    WZT_INTERIOR,
    WZT_INTERIOR_AL,
    WZT_TEST_LEVEL_AL,
    WZT_MISSION_AL,
    WZT_HIGH_LOD,
    NUM_WORLD_ZONE_TYPES
    };
    ```
    > [V] Knobby said:  
    > Some information about these: We have 2 distinct types of worlds that we load in SR3/4. We have a test level, which is a single zone of arbitrary size and then we have a streaming level which is broken up into pieces. We also have a global always loaded(WZT_ALWAYS_LOADED) named after the world and then individual always loaded files for each zone(WZT_STREAMING_AL). Missions and activities have a big "zone" that is loaded when the mission or activity is active. They can also have an always loaded portion.

??? quote "Position offsets"
    > [V] Knobby said:  
    >>    Quantum said:
    >> The coordinates in the world zone mesh file references are of type "et_int16" (16 bit values). I am displaying them as integers, but is there a factor I can apply that will translate them to standard game coordinate units?
    > 
    > We use the following to get an absolute position. Since the position is inside the zone, we add the reference position to it to get world offset:
    > 
    >```cpp
    > // transform of the instance
    > fl_matrix43 transform;
    > ref->m_transform.get(transform);
    > transform.m_translation += header->m_file_reference_offset
    >```


In the zone file itself we find a chunked format. It is a series of sections of data each with a header that describes the content. A section header looks like this:
```cpp
struct zone_section {
    et_uint32   m_id; // highest bit is a flag that tells us if the section has a gpu size. This flag is masked off after storing if the gpu size is to be expected.
    et_uint32   m_cpu_size; // cpu size of this section (does not include the header which is variable size)
    et_uint32   m_gpu_size; // if (gpu flag set
}
```

```cpp
struct wz_file_reference {
et_int16 m_pos_x;
et_int16 m_pos_y;
et_int16 m_pos_z;
et_int16 m_pitch;
et_int16 m_bank;
et_int16 m_heading;

// offset into v_file_header string pool for the filename
et_uint16 m_str_offset;
}

```

```cpp
Section ids:
0x2233 - crunched reference geometry - transforms and things for level meshes
0x2234 - objects - nav points, environmental effects, and many, many more things
0x2235 - navmesh
0x2236 - traffic data
0x2237 - world editor generated geometry - things directly made from the editor like terrain
0x2238 - sidewalk data
0x2239 - section trailer (??)
0x2240 - light clip meshes
0x2241 - traffic signal data
0x2242 - mover constraint data
0x2243 - zone triggers(interiors, missions)
0x2244 - heightmap
0x2245 - cobject rbb tree - cobjects are things that are not a full on object like tables and chairs
0x2246 - undergrowth - foliage
0x2247 - water volumes
0x2248 - wave killers
0x2249 - water surfaces
0x2250 - parking data
0x2251 - rain killers
0x2252 - level mesh supplemental lod data
0x2253 - cobject grid data - object fading
0x2254 - ae rbb (??)
0x2255 - havok pathfinding data(SR4 only?)
```

Object Section Data  
Object header data on top:
```cpp
uint32  signature           // 0x574F4246
uint32  version             // 5
int32   num_objects
int32   num_handles
uint32  flags
uint64  *handle list        // run-time
uint8   *object data        // run-time
uint32  object data size    // run-time
```

Header is followed immediately by handle list(64 bit handles), which is followed by object data.

The data is a series of property blocks:
```cpp
uint32  *handle                 // offset on disk
uint32  *parent handle          // offset on disk
uint32  object type hash        
uint16  number_of _properties
uint16  buffer size
uint16  name offset             // offset into the property list for the name of the object
uint16  padding

```
rest of the data is the property list itself. This is different data for each object, but the property list is just:
```cpp
uint16 type
uint16 size
uint32 name crc
```
followed by data. We store enumerations, flag data, strings, binary buffers, transforms, and other things in here and each object type looks for specific data based on that object.

type values are:
```cpp
0 - string
1 - data
2 - compressed transform - pos only(fl_vector)
3 - compressed transform with quaternion orient (fl_vector pos followed by fl_quaternion for orient)
```

fl_quaternion is a fancy want to compress data. It is a x, y, z, w float that represents a roation.

