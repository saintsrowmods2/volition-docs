# Chunks
!!! note "File extensions"
    `.chunk_pc`
    `.g_chunk_pc` 
    
## About

!!! info "[V] Knobby said:"
    ... The chunk_pc file is a loadable part of the world. It is what is needed for a physical chunk of the world. This includes the level geometry, level collision, sidewalk data, traffic splines, world objects, textures, etc. I say etc, but it really is probably just the tip of the iceberg. This is a massive monolithic file full of all kinds of things.

!!! note "File contents"
    __Things found in chunkfiles:__
    
    - **Materials**  
    Texture list, Shader params, Hashed shader names
    - **GPU Models**  
    Every visible model is in the g_chunk file. The format for static models is figured out.
    - **CPU Models**  
    The chunk_pc file itself contains models too, with no UV's or other extra data. Labeled as physmodels, since they're probably collision for loose objects.
    - **Objects**  
    Static world models and loose objects.
    - **Static world collision**  
    
    - **Light sources**
    - **Mesh movers**  
    Used for doors and other simple animations.

    __Not yet found but expected:__
    
      - Traffic paths, sidewalk data
    
    Some of this stuff could lie in the parts where the layout is already figured out. Just have to mess around with these values.

The .chunk_pc file internally shares a lot of structures with other files.


## .chunk_pc Layout

### header
256B header

??? note "Header"

    | Offset | Type | Name             | Comment                       |
    | ------ | ---- | ---------------- | ----------------------------- |
    | 0x0    | u32  | magic            |                               |
    | 0x4    | u32  | version          | always 121                    |
    | 0x8    |      |                  |                               |
    | 0xC    | u32  | (unused)         | always 0                      |
    | 0x10   |      |                  |                               |
    | ...    | ...  | ...              | ...                           |
    | 0x94   | u32  | cityobject_count |                               |
    | 0x98   | u32  | unknown23_count  |                               |
    | 0x9C   |      |                  |                               |
    | 0xA0   |      |                  |                               |
    | 0xA4   |      |                  |                               |
    | 0xA8   |      |                  |                               |
    | 0xAC   |      |                  |                               |
    | 0xB0   |      |                  |                               |
    | 0xB4   | u32  | mesh_mover_count |                               |
    | 0xB8   | u32  | unknown27_count  |                               |
    | 0xBC   | u32  | unknown28_count  |                               |
    | 0xC0   | u32  | unknown29_count  |                               |
    | 0xC4   | u32  | unknown30_count  |                               |
    | 0xC8   | u32  | unknown31_count  |                               |
    | 0xCC   |      |                  |                               |
    | 0xD0   |      |                  |                               |
    | 0xD4   | f32  |                  | These look like world coords. |
    | 0xD8   | f32  |                  |                               |
    | 0xDC   | f32  |                  |                               |
    | 0xE0   | f32  |                  |                               |
    | 0xE4   | f32  |                  |                               |
    | 0xE8   | f32  |                  |                               |
    | 0xEC   | f32  |                  |                               |
    | 0xF0   |      |                  |                               |
    | 0xF4   |      |                  |                               |
    | 0xF8   |      |                  |                               |
    | 0xFC   |      |                  |                               |

``` cpp title="chunk_pc header"
// size: 0x100
struct chunk_header {
    int32_t     signature;      //
    int32_t     version;        // Version 121
    int32_t     unknown0x08;    // 
    int32_t     unused_0x0c;    // always 0
    // ... todo ... //
}

```

### texture_list
Filename of each texture used in the chunk.
Always starts at 0x100 as the header is fixed length.
Size 16-byte aligned

``` cpp title="chunk_pc texturelist"

int32_t     num_textures;               //
uint32_t    padding[num_textures];      //
char        texture_names[num_textures];//
align 16                                //
```
The objects are split into two structs, located very far apart in the file. The second part is found at [#objects](.#objects).
Some of these transforms are left unused by objects.

### model_info
``` cpp
object_data_0_header    header
align 16
gpu_model_unknown       [header.num_gpu_models]
align 16
object_transform        [header.num_object_transforms]
align 16
object_unknown3         [header.num_unknown3s]
align 16
object_unknown4         [header.num_unknown4s]
align 16

```
``` cpp title="structs"

// size: 0x20
struct object_data_0_header {
    uint32_t    num_gpu_models;
    uint32_t    num_object_transforms;
    uint32_t    num_cpu_models;
    uint32_t    num_unknown3s;
    uint32_t    num_unknown4s;
};

// size: 0x18
struct gpu_model_unknown {
    int32_t unknown0x00;
    int32_t unknown0x04;
    int32_t unknown0x08;
    int32_t unknown0x0c;
    int32_t unknown0x10;
    int32_t unknown0x14;
};

// Every object has one of these, but there are always just a couple left unused??
// size: 0x50
struct object_transform {
    fl_vector   origin;         // XYZ 3x float
    fl_vector   xform_basis_x;  //
    fl_vector   xform_basis_y;  //
    fl_vector   xform_basis_z;  //
    // ... //
    float       unknown0x4C;    //
    uint32_t    unknown0x50;    // always 0?
    int32_t     unknown0x54;    //
    uint32_t    model_idx;      // gpu model index
    int32_t     unknown0x5C;    //
};

// size: 0x64
public struct object_unknown3 {
    float       unknown0x00;
    float       unknown0x04;
    float       unknown0x08;
    uint32_t    unknown0x0C;
    float       unknown0x10;
    uint32_t    unknown0x14;
    float       unknown0x18;
    float       unknown0x1C;
    float       unknown0x20;
    uint32_t    unknown0x24;
    float       unknown0x28;
    uint32_t    unknown0x2C;
    float       unknown0x30;
    float       unknown0x34;
    float       unknown0x38;
    float       unknown0x3C;
    float       unknown0x40;
    float       unknown0x44;
    float       unknown0x48;
    float       unknown0x4C;
    float       unknown0x50;
    float       unknown0x54;
    float       unknown0x58;
    float       unknown0x5C;
    uint32_t    unknown0x60;
};

// size: 0x34
public struct object_unknown4 {
    float   unknown0x00;
    float   unknown0x04;
    float   unknown0x08;
    float   unknown0x0C;
    float   unknown0x10;
    float   unknown0x14;
    float   unknown0x18;
    float   unknown0x1C;
    float   unknown0x20;
    float   unknown0x24;
    float   unknown0x28;
    float   unknown0x2C;
    float   unknown0x30;
};

```
### static_collision

??? note "links"
    _Some_ knowledge of havok collisions: [https://niftools.sourceforge.net/wiki/Nif_Format/Mopp](https://niftools.sourceforge.net/wiki/Nif_Format/Mopp)

It appears that the static meshes of the entire chunk are baked into one Havok collsion blob. This is inherently unmoddable, unless some genius comes and figures out Havok collisions.

The vertices seem to match visual models.


### model_buffer_headers

### materials
Very similar to already documented SRTT / IV format.

### g_model_headers

### objects

- objects
- names

``` cpp title="structs"

struct object {
    fl_vector   bb_min;             // bbox used for culling
    uint32_t    unknown0x0c;        // always 0?
    fl_vector   bb_max;             // bbox used for culling
    float       render_distance;    //
    uint32_t    unknown0x20;        // always 0?
    uint32_t    unknown0x24;        //
    uint32_t    unknown0x28;        // always 0?
    int16_t     unknown0x2C;        //
    int16_t     unknown0x2E;        // zero pad for alignemt
    int32_t     flags;              //
    int32_t     unknown0x34;        //
    int32_t     unknown0x38;        // Almost always 0. Exceptions: sr2_skybox, sr2_intdkmissunkdk, sr2_intarcutlimo, sr2_intaicutjyucar
    int32_t     unknown0x3C;        //
    uint32_t    transform_idx;      // See object_transform 
    uint32_t    unknown0x44;        //
    uint32_t    unknown0x48;        //
    uint32_t    unknown0x4C;        //
};

```

### unknown_names_12

!!! note
    Are these the names of destroyable objects?

### unknown13

### unknown18

!!! note
    These only appear in a couple chunkfiles around Saints HQ  
    See chunks numbered 93 and 106.

### unknown19

### unknown20

### unknown21

### unknown22

### unknown23

### unknown24

### unknown25

### mesh_movers

Count is found in the header.

### unknown27

Count is found in the header.

### unknown28

Count is found in the header.

### unknown29

Count is found in the header.

### unknown30

Count is found in the header.

### unknown31

Count is found in the header.

### unknown32

### mesh_mover_names

### lights

!!! warning
    The layout up to this point is only 99% figured out. Sometimes the lights aren't there.  
    If `num_lights == 'MCHK'`, stop. This bubblegum fix will save you from parsing garbage on _almost_ every chunk.

``` cpp
uint32_t    num_lights
uint32_t    unknown26b
light       lights[num_lights]
strz        light_names         //null terminated strings

align 16
float       light_unk2[]        // times num_unk_floats for every light
align 16


```
``` cpp title="structs"

struct light {
    int32_t     flags;
    uint32_t    unknown0x04;    // always 0?
    fl_vector   color;          // RGB 3x float
    uint32_t    unknown0x14;
    uint32_t    unknown0x18;
    uint32_t    unknown0x1c;
    uint32_t    num_unk_floats;
    int32_t     unknown0x24;    // always -1?
    int32_t     unknown0x28;    // Iggy's performance complaint
    int32_t     unknown0x2c;    // always -1?
    uint32_t    unknown0x30;
    float       unknown0x34;
    uint32_t    unknown0x38;
    fl_vector   origin;         // XYZ 3x float
    fl_vector   xform_basis_x;  // Basis X (probably)
    fl_vector   xform_basis_y;  // Basis Y (probably)
    fl_vector   xform_basis_z;  // Basis Z (probably)
    float       unknown0x6c;
    float       unknown0x70;
    float       radius_inner;   // Light has full intensity until this distance
    float       radius_outer;   // 
    float       render_dist;    //
    int32_t     unknown0x80;    // always -1?
    int32_t     parent_idx;     // -1 if none. Origin is relative to parent object, if set.
    uint32_t    unknown0x88;    // 
    uint32_t    unknown0x8c;    //
    uint32_t    type;           // ? maybe? Possible values: 0, 1, 2, 3
};

```
``` cpp title="Flags"
    // These flags are present at least sometimes.
    // TODO: verify that there aren't more of them
    // The names of shadow / light flags were found in the game exe, are they
    // exposed to lua?
    FLAG_UNKNOWN0 = 0x8;
    FLAG_UNKNOWN1 = 0x10;
    FLAG_UNKNOWN2 = 0x20;
    FLAG_UNKNOWN3 = 0x40;
    FLAG_UNKNOWN4 = 0x80;
    FLAG_LIGHT_LEVEL = 0x100;       // Cast light on static
    FLAG_LIGHT_CHARACTER = 0x200;   // Cast light on characters and props
    FLAG_SHADOW_LEVEL = 0x400;      // Static meshes create shadows
    FLAG_SHADOW_CHARACTER = 0x800;  // Characters and props create shadows
    FLAG_UNKNOWN9 = 0x2000;
    FLAG_UNKNOWN10 = 0x8000;
    FLAG_UNKNOWN11 = 0x20000;
```