# Animation Formats
??? note "Links"
    - [[V] RabbleRooster | Crunched Animation Formats](https://www.saintsrowmods.com/forum/threads/crunched-animation-formats.15963/)
??? note "Tools"
    -
___


[QUOTE="[V] RabbleRooster, post: 100706, member: 44315"]
This post is like the mesh one, except I *should* understand these better (although, quite frankly, everything has changed since then). 

Rigs are the listing of bones, their parents, and the positional relationship in a T-pose (there's no rotation data in a crunched rig):
```cpp
struct anim_tag {
    V_FILE_PTR(char *)    name;
    et_fl_matrix43            transformation;
    et_int                    parent_index;
    et_int                    vid;
};

// base bone
class anim_bone {
public:
    V_FILE_PTR(char *)    name;
    et_fl_vector            inv_translation;        //Relative to the skin pivot
    et_fl_vector            rel_bone_translation;    //Relative to the parent
    et_int                    parent_index;
    et_int                    vid;
};

class anim_rig {
public:   
    char                name[ANIMLIB_MAX_ANIM_RIG_NAME_LEN];
    et_uint            flags;
    et_int            num_bones;
    et_int            num_common_bones;
    et_int            num_virtual_bones;
    et_int            num_tags;

    V_FILE_PTR(uint32 *)    bone_name_chksums;
    V_FILE_PTR(anim_bone    *) bones;
    V_FILE_PTR(anim_tag *)  tags;
};
```


Here's the header for the anim file:
```cpp
#define AFF_PACKED_KEYS                    (1<<0) // tightly packed rotation keys
#define AFF_TIME_AS_SHORTS                (1<<1) // time is stored as shorts instead of chars
#define AFF_DELTA_TIMES                    (1<<2) // animation times are stored as deltas
#define AFF_HAS_MOTION                    (1<<3) // animation has motion information
#define AFF_HAS_MORPH                    (1<<4) // animation has morph information
#define AFF_LONG_TRANSLATIONS            (1<<5) // animation has animated bones that extend beyond 512 meters
#define AFF_HAS_WEIGHT_KEYS            (1<<6) // animation has weight keys
#define AFF_SHORT_COUNTS                (1<<7) // animation has more than 255 rotation or translational keyframes
#define AFF_HAS_SCALE_KEYS                (0) // animation has scale keys

struct anim_file {
    et_int32        id;
    uint8            version;
    uint8            flags; // see AFF_* above
    et_uint16    end_frame;

    uint8            ramp_in;                        // ramp in  (in MAX ticks)
    uint8            ramp_out;                    // ramp out (in MAX ticks)
    uint8            num_bones;                    // number of animated bones (those with keyframes - even if identity)
    uint8            num_rig_bones;                // num_rig_bones when anim was crunched.

    et_fl_quaternion        total_rotation;
    et_fl_vector            total_translation;

    V_FILE_PTR(uint8 *) anim_to_rig_bone_mapping;    // array of size ANIM_LIB_CFG_MAX_BONES
    V_FILE_PTR(uint8 *) root_controller_bone;        // pointer to the bone data for the root controller.  All other bones have to be stepped through in code
    uint8            data[0];
};
```

The actual animation key data is packed in some really funky ways (spelling mistakes and all).  Here are the relevant types:
```cpp
struct char_quat {
    int8 x, y, z;
};

// a packed rotation offset
// a b and c components represent the 3 smallest values of a quaternion.
#if defined(__PC)
struct afc_rotation_offset {
    int8 a                    : 6;
    uint8 a_scaler_bits    : 2;
    int8 b                    : 6;
    uint8 b_scaler_bits    : 2;
    int8 c                    : 6;
    uint8 c_scaler_bits    : 2;

    uint8 count                : 6;
    uint8 largest            : 2;
};
#else
struct afc_rotation_offset {
    uint8 a_scaler_bits    : 2;
    int8 a                    : 6;
    uint8 b_scaler_bits    : 2;
    int8 b                    : 6;
    uint8 c_scaler_bits    : 2;
    int8 c                    : 6;

    uint8 largest            : 2;
    uint8 count                : 6;
};
#endif

// data is packed into 4 6 bit values.
// easiest way to retrieve is to cast data to an int pointer and copy, then do manipulation
// xbox2:    | x:6 bits | y:6 bits | z:6 bits | time:6 bits | junk: 8 bits |
// pc/ps3:    | junk: 8 bits | x:6 bits | y:6 bits | z:6 bits | time:6 bits |

// byte 0 : | x:6  |y:2|
// byte 1 : | y:4 | z:4|
// byte 2 : |z:2|time:6|

// in order to preserve the sign bit, first shift all the bits to the far left before shifting 26 bits back to the far right. (26 = 32 - 6)
class afc_rotation_key_packed {
private:
    uint8 data[3];
public:
    inline int get_time() {
        return (data[2] & 0x3f);
    };
    inline int get_a() {
        return (((int)(data[0] << 24)) >> 26);
    };
    inline int get_b() {
        return (((int)((data[0] << 30) | ((data[1] & 0xf0) << 22))) >> 26);
    };
    inline int get_c() {
        return (((int)((data[1] << 28) | ((data[2] & 0xc0) << 20))) >> 26);
    };

};

struct afc_rotation_key {
    char_quat rotation;
};

// in millimeter units, so a char can represent +/- 127 millimeters
struct afc_translation_key {
    char x;
    char y;
    char z;
};

// in 1 / 64 meter units, so a short can represent +/- 512 meters
struct afc_translation_offset {
    uint16 num_keys;
    int16 x;
    int16 y;
    int16 z;
};

struct afc_translation_offset_long {
    uint32 num_keys;
    int32 x;
    int32 y;
    int32 z;
};

struct afc_scale_key {
    fl_vector scale;
};

struct afc_weight_key {
    float weight;
};

struct afc_morph_key {
    uint8 weight;
};
```

If there is interest, I can do a bit to demystify the storage of all this data in the future.


Here's the data structures for cloth sim files (there may be some header before the main cloth_sim_info struct):
 
```cpp
struct simulated_node_info {
    int8 bone_index;
    int8 parent_node_index;    // rotation is applied to the parent bone for each particle
    int8 gravity_link;
    int8 anchor;

    uint32 collide;        // binary list of colliders that this node collides with.

    float wind_multiplier;
    xxx_vector_conv pos;
    xxx_vector_conv local_space_pos;
};

struct simulated_node_link_info {
    uint16 node_index_1;
    uint16 node_index_2;

    uint32 collide;        // bit list of colliders that this linke collides with

    float    length;
    float stretch_len;
    float twist;
    float spring;
    float damp;
};

struct cloth_sim_collision_primitive_info {
    int    bone_index;
    short    is_capsule;
    short    do_scale;
    float    radius;
    float    height;
    xxx_vector_conv pos;
    xxx_vector_conv axis;
    xxx_vector_conv local_space_pos;
};

struct cloth_sim_rope_info {
    float    length;
    int    num_nodes;
    int    num_links;
    int    *node_indecies;
    int    *link_indecies;
};

struct cloth_sim_info {
    int    version;
    int    data_size;
    char  name[28];
    int    num_passes;
    float    air_resistance;
    float wind_multiplier;
    float wind_const;   
    float    gravity_multiplier;
    float object_velocity_inheritance;
    float object_position_inheritance;
    float object_rotation_inheritance;
    int    wind_type;

    int    num_nodes;
    int    num_anchor_nodes;
    int    num_node_links;
    int    num_ropes;
    int    num_colliders;

    simulated_node_info                *nodes;
    simulated_node_link_info        *node_links;
    cloth_sim_rope_info                *ropes;
    cloth_sim_collision_primitive_info *colliders;
};
```


Finally, I'll add morph information here:
```cpp
struct rl_morph_target_data {
    et_uint                                    name_chksum;
    et_uint                                    num_vbuffers;
    V_FILE_PTR(rl_morph_buffer_generic *) vbuffers; //type is given by target_format in parent structure
};

struct rl_morph_data {
    et_int                            signature;
    et_int                            version;

    et_int                            target_format;
    et_int                            num_targets;
    V_FILE_PTR(rl_morph_target_data *) targets;
};

class rl_morph_container : public rl_base_object {
protected:
    uint32    m_debug_signature;
    rl_morph_data                *m_morph_data;
};

struct mesh_morph {
    et_int                        signature;
    et_int                        version;

    V_FILE_PTR(rl_morph_container *)    morph_target;
};
```
