# .chunk_pc

## Info

### Location

- chunks1.vpp_pc
- chunks2.vpp_pc
- chunks3.vpp_pc

### Related
- [g_chunk_pc](g_chunk_pc)
- [g_peg_pc](g_peg_pc)
- [city_pc](city_pc])
  
## Layout

### Header
256B header

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
int32       signature;      //
int32       version;        // Version 121
int32       unknown_0x8;    // 
int32       unused_0xc;     // always 0

```

### Texture list
Filename of each texture used in the chunk.
Always starts at 0x100 as the header is fixed length.
Size 16-byte aligned

``` cpp title="chunk_pc texturelist"

int32       num_textures;               //
uint32      padding[num_textures];      //
char        texture_names[num_textures];//
<alignment padding 16>                  //

```