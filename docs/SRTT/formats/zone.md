# Zone files

https://www.saintsrowmods.com/forum/threads/sr3-zone-file-format.2855/

The zone file is split up into a header(czh file) and the zone data itself (czn file). We did this to save memory since we only need the header portion for building streaming containers and at load time. We load and then dump this header data.

There is a v_file_header on top of the czh that describes the things that the zone references. This will be things like textures and meshes and they end up in the str2 file.

```cpp 
v_file_header:
uint16 signature - 0x3854
uint16 version - 4
uint32 reference_data_size
uint32 reference_data_start(offset)
uint32 reference_count - number of references in the file
uint8 pad[16] - zero'd before write, padding so we can add to the header
```

Reference data always follows the header. That data starts at reference_data_start offset from the top of the v_file_header and to get to the next reference(string) you just advance past the string. Double null termination on this string list, but the size is known as well.

Immediately following the v_file_header is the world zone header:
```cpp
uint32 signature - 'Z3RS'
uint32 version - 29
v_file_header *ptr - 8 bytes on PC, points back up into the v_file_header since we use that at load time to find filenames(saving some space)
fl_vector file_reference_offset - applied to all file references to move them to this zone
wz_file_reference *m_file_references
uint16 num_file_references
uint8 zone_type
uint8 unused
interior_trigger *ptr - run-time data
uint16 number of triggers - run-time data
uint16 extra objects - only used in test levels(not sure for what)
uint8 pad[24] - zero'd before write, padding so we can add to the header
```

In the zone file itself we find a chunked format. It is a series of sections of data each with a header that describes the content. A section header looks like this:
uint32 id - highest bit is a flag that tells us if the section has a gpu size. This flag is masked off after storing if the gpu size is to be expected.
uint32 cpu size - cpu size of this section (does not include the header which is variable size)
if (gpu flag set) {
uint32 gpu_size
}

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

Object Section Data
Object header data on top:
uint32 signature - 0x574F4246
uint32 version - 5
int32 num_objects
int32 num_handles
uint32 flags
uint64 *handle list - run-time
uint8 *object data - run-time
uint32 object data size - run-time
```

Header is followed immediately by handle list(64 bit handles), which is followed by object data.

The data is a series of property blocks:
```cpp
uint32 *handle - offset on disk
uint32 *parent handle - offset on disk
uint32 object type hash
uint16 number_of _properties
uint16 buffer size
uint16 name offset - offset into the property list for the name of the object
uint16 padding
rest of the data is the property list itself. This is different data for each object, but the property list is just:
uint16 type
uint16 size
uint32 name crc
```
followed by data. We store enumerations, flag data, strings, binary buffers, transforms, and other things in here and each object type looks for specific data based on that object.

type values are:
0 - string
1 - data
2 - compressed transform - pos only(fl_vector)
3 - compressed transform with quaternion orient (fl_vector pos followed by fl_quaternion for orient)

fl_quaternion is a fancy want to compress data. It is a x, y, z, w float that represents a roation.