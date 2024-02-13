# Pegfiles
??? note "Links"
    - https://www.saintsrowmods.com/forum/threads/peg-file-format.2908/
??? note "Tools"
    -
___

Pegleg image format ⛵🏴‍☠️


```cpp title="peg_header"
struct VLIB_EXPORT peg_header {
   int32      signature;
   int16      version;
   int16      platform;            // see peg_platform enum
   int32      dir_block_size;         // calculated by bm_save_peg(). includes size of header, plus padding.
   int32      data_block_size;       // calculated by bm_save_peg(). includes padding the allocator might have done.
   int16      num_bitmaps;
   int16      flags;
   int16      total_entries;
   int16      align_value;
   peg_entry  peg_entries[0];          // Can use this to access the entries

};

```


Then the peg entry itself(one in a vbm file, multiple in a peg)

```cpp title="peg_entry"
struct peg_entry {
 uint8         *data;  // written to disk as offset from start of file.
 uint16        width;
 uint16        height;
 uint16        bm_fmt;
 uint16        pal_fmt;
 uint16        anim_tiles_width;  // for animated textures using an anim sheet BM_F_ANIM_SHEET
 uint16        anim_tiles_height;
 uint16        num_frames;
 uint16        flags;        // see BM_F_* defines   
 char          *filename;
 uint16        pal_size;
 uint8         fps;
 uint8         mip_levels;  // Base frame + number of mipmaps (always at least 1)
 uint32        frame_size;  // Bytes in palette + image.
 peg_entry     *next;        // each base bitmap_entry will maintain a linked list of
 peg_entry     *prev;        // actual peg entries so we can do unloading of pegs very quickly
 uint32        cache[2];    // generic texture caching data, used differently on different platforms

};

```



```cpp title="Flags"

#define BM_F_ALPHA              (1<<0)  // bitmap has alpha
#define BM_F_NONPOW2            (1<<1)  // bitmap is not power of 2
#define BM_F_ALPHA_TEST         (1<<2)
#define BM_F_CUBE_MAP           (1<<3)  // bitmap is a cube map, react appropriately on load.
#define BM_F_INTERLEAVED_MIPS   (1<<4)  // bitmap contains interleaved mips (they exist inside of the NEXT bitmap)
#define BM_F_INTERLEAVED_DATA   (1<<5)  // bitmap contains interleaved mips from the previous bitmap
#define BM_F_DEBUG_DATA_COPIED  (1<<6)  // used by the peg assembler only.
#define BM_F_DYNAMIC            (1<<7)  // bitmap was loaded dynamically (not from a peg) (runtime only)
#define BM_F_ANIM_SHEET         (1<<8)  // bitmap animation frames are stored in one bitmap spaced sequentially left to right
#define BM_F_LINEAR_COLOR_SPACE (1<<9)  // bitmap is NOT stored in SRGB space, it is linear
#define BM_F_HIGH_MIP           (1<<10) // bitmap is a separately streamed high mip
#define BM_F_HIGH_MIP_ELIGIBLE  (1<<11) // bitmap is eligible for linking up with a high mip (runtime only flag)
#define BM_F_LINKED_TO_HIGH_MIP (1<<12) // bitmap is currently linked to a high mip (runtime only flag)
#define BM_F_PERM_REGISTERED    (1<<13) // bitmap is permanently registered. used on the PC so d3d becomes the permanent owner of the texture memory.

```




```cpp title="bm_fmt values of interest"
   BM_FORMAT_PC_DXT1                                = 400,
   BM_FORMAT_PC_DXT3,
   BM_FORMAT_PC_DXT5,
   BM_FORMAT_PC_565,
   BM_FORMAT_PC_1555,
   BM_FORMAT_PC_4444,
   BM_FORMAT_PC_888,
   BM_FORMAT_PC_8888,
   BM_FORMAT_PC_16_DUDV,
   BM_FORMAT_PC_16_DOT3_COMPRESSED,
   BM_FORMAT_PC_A8,

```