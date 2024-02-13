# Mesh Formats
??? note "Links"
    - [[V] RabbleRooster | Crunched Mesh Formats](https://www.saintsrowmods.com/forum/threads/crunched-mesh-formats.15962/)
??? note "Tools"
    -
___

cmesh and smesh share the same structure, called static_mesh (not confusing). The cpu file gets a [`v_file_header`](/SRTT/formats/common/#v_file_header), followed by:

 - static_mesh
 - Texture names
 - Navpoints
 - Collision spheres
 - Collision cylinders
 - Rig bone indices
 - Mesh data
 - Materials
 - VIDs
 - submesh LOD info

The relevant structures for static_mesh:
```cpp
// Navigation reference point
struct static_mesh_navp {
    char                            name[MAX_NAVP_NAME_LENGTH];            // name to reference nav point by
    et_int                        vid;                                            // vid this navp is attached to.
    et_fl_vector                pos;                                            // position of navpoint in object coords
    et_fl_quaternion            orient;                                        // quaternion representation of navpoint
};

struct cmesh_csphere {
    et_uint32        body_part_id;                // ID for the body part this represents
    et_int            parent_index;                // index of parent bone (-1 if none)     
    et_fl_vector    pos;                            // position of collision sphere in object coords 
    et_float            radius;                        // radius of collision sphere
};

struct cmesh_ccylinder {
    et_uint32        body_part_id;                // ID for the body part this represents
    et_int            parent_index;
    et_fl_vector    axis;
    et_fl_vector    pos;
    et_float            radius;
    et_float            height;
};

class static_mesh {
public:
    et_int                        signature;
    et_short                        version;
    et_short                        mesh_flags;

    // grouping the various count fields together to minimize padding waste
    et_short                        num_navpoints;
    et_short                        num_rig_bones;                                // Number of bones this character mesh uses from the RIG file.
    et_short                        num_materials;                                // number of entries in materials array
    et_short                        num_material_maps;                        // number of entries in material_maps and material_map_name_crcs arrays
    et_short                        num_lods_per_submesh; //number of lods per submesh. All submeshes must have same number of lods, currently.
    et_uint16                        num_submesh_vids;    //this will be >= num_logical_submeshes (there can be additional vids tacked on the end purely for informational reasons)

    V_FILE_PTR(static_mesh_navp *) navpoints;                                    // prop points, I think.

    V_FILE_PTR(et_uint *)    rig_bone_indices;                        // List of RIG bone indices used by this character mesh.

    et_fl_vector                bounding_center;                            // for some reason, these are only in VIF meshes.
    et_float                        bounding_radius;

    // collision primitives for inter-character hit detection
    et_short                        m_num_cspheres;                            // collider spheres used for hit detection
    et_short                        m_num_ccylinders;                            // collider cylinders used for hit detection
 
   // 4 BYTES PADDING IF V_FILE_PTR is 8 bytes
    V_FILE_PTR(cmesh_csphere *)    m_cspheres;
    V_FILE_PTR(cmesh_ccylinder *) m_ccylinders;

    // rendering mesh data
    V_FILE_PTR(rl_mesh *)         mesh;

    //NOTE: It is possible for material_maps and materials to be NULL. That means the materials are expected to be provided externally.
    V_FILE_PTR(V_FILE_PTR(rl_material_map*) *)       material_maps;                            // num_material_maps sized array of material maps
    V_FILE_PTR(et_uint32 *)                                 material_map_name_crcs;                // num_material_maps sized array of name crc for each material map
    V_FILE_PTR(V_FILE_PTR(rl_static_material *) *)   materials;                                // num_materials sized array of materials

    //logical number of submeshes (same as actual number of submeshes only if we don't have lods)
    et_uint32               num_logical_submeshes;

   // 4 BYTES PADDING IF V_FILE_PTR is 8 bytes

    //This pointer will either be null (no VID info) or will point to an array of ints of length num_logical_submeshes
    //It will contain the VID corresponding to each logical submesh
    V_FILE_PTR(et_uint32 *) submesh_vids;

    //This pointer will either be null (no LOD info) or will point to a (flat) 2D array, representing the rl_mesh submesh indexes for each lod level of each "logical submesh"
    //submesh_lod_info[logical_submesh_index*num_lods_per_submesh + lod_level] --> index into rl_mesh's flat array of submeshes.
    V_FILE_PTR(et_short *)  submesh_lod_info;
};
```


The CPU mesh data itself has a little header information:
[LIST]
[*]Version (uint16 + 4 byte pad)
[*]CRC to validate mapping with the GPU file (uint32)
[*]CPU size (uint32)
[*]GPU size (uint32)
[/LIST]
Then we get the data:

```cpp
struct rl_vertex_buffer_data
{
   et_int32                       num_verts;           // number of vertices in the buffer
   uint8                           vert_stride_0;       // stride (size) of each vertex (stream 0)
   uint8                           vertex_format;       // vertex format
   uint8                           num_uv_channels;   // number of uv channels (from 0 to 4)
   uint8                           vert_stride_1;       // stride (size) of each vertex (stream 1)

   V_FILE_PTR(void *)      render_data;     // pointer to platform specific data we need for this vertex buffer (created at run-time)
   V_FILE_PTR(rl_mesh_generic_vertex *) verts;
};

struct rl_index_buffer_data
{
    et_uint32                    num_indices;            // number of indices in the index buffer
   // 4 BYTE PAD WHEN V_FILE_PTR IS 8 BYTES
    V_FILE_PTR(rl_generic_index *) indices;        // can be either et_uint16 or et_uint32
    uint8                            index_size;                // size of indices in bytes (2 or 4)
    uint8                            prim_type;                // keep it 4 byte aligned
    et_uint16                    num_blocks;
};

struct rl_bone_group_data
{
    uint8                            num_group_bones;        // How many entries in the mapped_bone_list this group uses.
    uint8                            group_bone_offset;    // Index in the mapped_bone_list of the first bone in this group.
};

struct rl_bone_map_data
{
    et_uint16                    num_mapped_bones;
   // 2 BYTES OR 6 BYTES PAD DEPENDING ON V_FILE_PTR SIZE
    V_FILE_PTR(uint8 *)        mapped_bone_list;

    et_uint16                    num_bone_groups;
   // 2 BYTES OR 6 BYTES PAD DEPENDING ON V_FILE_PTR SIZE
    V_FILE_PTR(rl_bone_group_data *) bone_group_list;
};

struct rl_mesh_data
{
    et_uint32                            mesh_flags;
    et_int32                                num_sub_meshes;
    V_FILE_PTR(rl_submesh_data *) sub_meshes;
    et_uint32                            num_vertex_buffers;
   // 4 BYTE PAD WHEN V_FILE_PTR IS 8 BYTES
    V_FILE_PTR(rl_vertex_buffer_data *) vertex_buffers;
    rl_index_buffer_data                index_buffer;
    rl_bone_map_data                    bone_map;

    //                these were moved out of vertex buffer data structure when we put in
    //                multiple vertex formats, because it does not make sense to have different
    //                scales/offsets for different vertex buffers on the same mesh.
    et_fl_vector                        position_scale;    // scale to apply to vertex positions
    et_fl_vector                        position_offset;    // offset to apply to vertex positions
};
```


Vertex format will be one of these (most likely float3 or xcposition):
```cpp
enum rl_vertex_attribute_type {
    RLVA_INVALID_TYPE = -1,

    ////////////////////////
    // Floating Point Types
    RLVAT_FLOAT1 = 0,
    RLVAT_FLOAT2,
    RLVAT_FLOAT3,
    RLVAT_FLOAT4,

    ////////////////////////
    // Half Float Types
    RLVAT_HALF2,
    RLVAT_HALF4,

    ////////////////////////
    // Byte Types
    RLVAT_UBYTE4,
    RLVAT_UBYTE4N,
   
    ////////////////////////
    // Short Types
    RLVAT_SHORT2N,
    RLVAT_SHORT4N,
    RLVAT_SHORT2,
    RLVAT_SHORT4,

    ////////////////////////
    // Compressed Normal Meta Types
    RLVAT_CNORMAL,
    RLVAT_CTANGENT,

    ////////////////////////
    // Color Meta Types
    RLVAT_COLOR4,

    ////////////////////////
    // Compressed Position Meta Types
    RLVAT_CPOSITION,
    RLVAT_XCPOSITION,

    NUM_RL_VERTEX_ATTRIBUTE_TYPES
};
```

lmesh files are for level meshes. They also get a v file header, then this header structure:
```cpp
struct level_mesh_aux_data {
    et_fl_bbox                    m_bbox;

    // these are flags stored about the entries. There are num_prims/2 bytes in this array.
    // Each nibble has the information for a single primitive with the least
    // significant bit having the first primitive's flags, ie, ((m_flags[i]) & 0xF) = first primitive
    // (flag & 0xF0) >> 4 = second primitive
    uint8                            m_flags[0];
};

struct level_mesh_subinstance {
public:
    et_fl_matrix43          m_transform;
    et_uint16               m_submesh_index;
    et_uint16               m_flags;
};

struct level_mesh_zbias_decal {
public:
    et_fl_matrix43                m_transform;
    et_int                        m_decal_layer;
    et_uint32                    m_mesh_index;
};

struct level_mesh_stitch_edge {
public:
    et_uint32            m_num_knots;
    V_FILE_PTR(et_fl_vector *)    m_knots;
};

struct level_mesh_posdummy {
    et_int               m_type;       // use level_mesh_posdummy_types
    et_fl_matrix43       m_transform;
};

struct level_mesh_occluder {
    et_vm_matrix44 m_transform;
    et_vm_bbox m_aabb;
};

struct level_mesh_lod_array {
    static const int max_lods = 3;
    level_mesh_lod_array()
    {
        for (int i = 0; i < max_lods; i++) {
            m_lods[i] = NULL;
            m_variant_values[i] = NULL;
        }
        m_level_mesh_name[0]=0;
    }
    rl_multi_mesh_instance::lod* m_lods[max_lods];
    level_mesh_variant_value *m_variant_values[max_lods];
    linked_list_p<level_mesh_header> m_level_meshes[max_lods];

    // debug cursor (which also works in release) name
    static const int max_name_len = 48;
    char m_level_mesh_name[max_name_len];
};

struct level_mesh_material_variant {
    et_uint32 m_material_name_checksum;
    et_uint16 m_variant_0_id;
    et_uint16 m_variant_1_id;
};


class level_mesh_header {
public:
    et_uint32                    m_signature;
    et_uint16                    m_version;

   // 2 BYTES PADDING
    et_uint32                    m_flags;

    et_fl_vector                m_bounding_center;                         
    et_float                        m_bounding_radius;

    et_vm_bbox                    m_local_aabb;

    // :ANDYC:TODO: remove when zbias decals are part of multi mesh instance

    // :BEGIN REMOVE LATER:
    struct {
        // rendering mesh data
        // 4 BYTES PADDING IF V_FILE_PTRS are 8 bytes
        V_FILE_PTR(V_FILE_PTR(rl_mesh *)*)    m_base_meshes;
        et_uint32                    m_num_base_meshes;

        V_FILE_PTR(V_FILE_PTR(rl_material_map *) *)    m_base_material_maps;
        et_uint32                    m_num_base_material_maps;

        // 4 BYTES PADDING IF V_FILE_PTRS are 8 bytes
        V_FILE_PTR(const char *)    m_texture_names;

        V_FILE_PTR(level_mesh_zbias_decal *)    m_zbias_decals;
        et_uint32                    m_num_zbias_decals;
 
    } m_decals;
    // :END REMOVE LATER:


    et_int32                        m_cm_index;
    et_int32                        m_corpse_cm_index;
    V_FILE_PTR(hkpShape*)    m_static_collision_model;

    et_int32                        m_num_stitch_edges;
    V_FILE_PTR(level_mesh_stitch_edge *) m_stitch_edges;

    et_int32                m_num_posdummies;
    // 4 BYTES PADDING IF V_FILE_PTRS are 8 bytes
    V_FILE_PTR(level_mesh_posdummy *) m_posdummies;

    et_int32                m_num_occluders;
    // 4 BYTES PADDING
    V_WIDE_PTR(level_mesh_occluder *) m_occluders;  // Using a wide ptr here to make decrementing pad bytes simpler.

    V_FILE_PTR(rl_multi_mesh_data*) m_multi_mesh_lod;

    et_fl_vector                m_collision_bmin;
    et_fl_vector                m_collision_bmax;

    // default values for when no variants are provided
    et_uint                        m_num_material_variants;
    V_WIDE_PTR(level_mesh_material_variant *) m_material_variants;
    et_uint                        m_num_variant_values;
    V_WIDE_PTR(level_mesh_variant_value *) m_variant_values;

    // crc maps for each lod
    et_uint                        m_this_lod_map_index;
    et_uint                        m_num_variant_crc_maps;
    V_WIDE_PTR(et_uint*)        m_variant_crc_maps_counts;                     
    V_WIDE_PTR(V_WIDE_PTR(et_uint32 *)*) m_variant_crc_maps;

    // for keeping track to duplicate references in the level_mesh_lod_array code
    V_WIDE_PTR(level_mesh_header*) next;
    V_WIDE_PTR(level_mesh_header*) prev;

    volatile uint32            m_preload_done;
    uint8                            m_pad_bytes[16];        // will always be zeroed.  Use this space for easier versioning of header.
};
```


matlib files have a v file header, then this header:
```cpp
struct material_library {
    et_uint32                    m_signature;
    et_uint32                    m_version;

    // RL data
    V_FILE_PTR(rl_material_map *) m_material_map;
    V_FILE_PTR(V_FILE_PTR(rl_static_material *) *) m_materials;
    et_uint                                m_num_materials;

   // 4 BYTES PAD IF V_FILE_PTRS ARE 8 BYTES

    V_FILE_PTR(char const *)        m_texture_names;

    // in the 1:1 file the following are tagged on
    // << material data
};
```


The actual material data structure looks like this:
```cpp
struct rl_material_const {
    et_float                    elem[4];
};

// 1:1 data structure for writing texture descs to disk
//
struct rl_material_texture_desc {
    rl_texture_handle        texture_handle;
    et_uint32                name_checksum; //the crc_stri() of the name of the texture semantic in the shader file, not the texture filename
    et_uint16                texture_stage;
    et_uint16                texture_flags;
};

// 1:1 data structure for writing materials to disk
//
struct rl_material_data {
    rl_shader_handle                shader_handle;
    et_uint32                        name_checksum;

    // flags for the material see the RLMF_* defines
    et_uint32                        mat_flags;

    // packed into 4 bytes
    et_uint16                        num_textures;
    uint8                                num_constants; // the number of quadwords in the constant block
    uint8                                max_constants; // the actual space allocated (extra space for adding constants in shaders at runtime)

    V_FILE_PTR(rl_material_texture_desc *)    textures;

    V_FILE_PTR(et_uint32 *)                constant_name_checksum; // array of crc`s of the constant names in the shader file
    V_FILE_PTR(rl_material_const *)    constant_block;             // the pointer to the block of data that get's upload to the user constants

    //                    There is no need to store the alpha version of a shader in a material... a material is either alpha or not alpha.  So in the case of an alpha
    //                     material, the shader handle will point to the alpha version of the inferred lighting shader.  Only keeping the below field around until the next
    //                     global recrunch.
    rl_shader_handle                alpha_shader_handle;         // shader handle for alpha version of shader (only used with inferred rendering)
};
```
