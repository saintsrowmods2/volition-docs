https://www.saintsrowmods.com/forum/threads/asm-and-vpp-file-formats.8448/


# Packfiles and ASM

## Packfile
Packfiles consist of a big header on top followed by invdividual entries. The header is actually different pieces aligned to 2048 bytes for historical cd drive reasons. We start with the v_packfile structure itself followed by the directory(v_packfile_entries) and that is followed by the filename list. We convert the pointers to offsets into the filename pool when writing to disk.

```cpp 
struct v_packfile_entry {
    char                        *filename;         // filename

    uint32                      sector;            // sector offset RELATIVE to the start of the packfile
    uint32                      start;             // offset from start of v_packfile::data (if data is valid)

    uint32                      size;              // file size
    uint32                      compressed_size;   // compressed file size
    struct v_packfile           *parent;           // my parent
};
```

```cpp
struct v_packfile {
    uint32            descriptor;        // packfile descriptor used to validate data
    uint32            version;           // version number of packfile
    char              short_name[V_MAX_PACKFILE_NAME_LEN+1];  // filename - 8.3 format
    char              pathname[V_PACK_MAX_PATH_LEN+1];        // pathname

    uint32            flags;             // packfile flags
    uint32            sector;            // packfile starts at this sector

    uint32            num_files;         // number of files in *data section
    uint32            file_size;         // physical size (in bytes) of the source vpp file
    uint32            dir_size;          // number of bytes in directory section
    uint32            filename_size;     // number of bytes in filename section

    uint32            data_size;         // number of uncompressed bytes in data files
    uint32            compressed_data_size; // number of compressed bytes in *data section

    v_packfile_entry  *dir;              // directory section
    char              *filenames;        // file name section
    uint8             *data;             // data section -- set gameside when a packfile is wholely loaded into memory (temp, condensed, or memory mapped)

    uint32            open_count;        // how many files have open handles into the packfile
};
```

## ASM

This is the file for streaming purposes that describes the resources required to load a "thing".

We start with a container(thing) header:
```cpp 
struct stream2_container_header {
     uint32    signature;
    uint16    version;
    uint16    num_containers;   
};
``` 
This is followed by some mappings for allocators, primitives(files), and containers so we can move things around internally and still match up to the enums:
```cpp 
int32  num_allocator_remaps
  char *name
  uint8 stored_index

int32  num_primitive_remaps
  char *name
  uint8 stored_index

int32  num_container_remaps
  char *name
  uint8 stored_index
```

We then read each container from the asm file and they look like this when stored:
```cpp 
struct stored_container {
  char   *name;
  uint8  container_type;
  uint16 load_flags;
  int16  num_entries;
  uint32 packfile_base_offset;
  char   *stub_parent_name;
  int32  aux_data_size;
  uint8  *aux_data;
  int32  packfile_total_compressed_read_size;
}
```
This is followed by num_entries * 2 uint32's which are the file sizes for the entries in the packfile.
Next we spin through each entry and load it:
```cpp 
struct stored_primitive {
  char  *name;
  uint8 primitive_type;
  uint8 allocator;
  uint8 flags;
  uint8 split_extension_index;
  int32 cpu_size;
  int32 gpu_size;
  uint8 allocation_group;
}
```

## Notes:

 - filename like strings are usually stored by writing the length of the string as a uint16 and then the string bytes themselves.
 - a primitive type can have multiple extensions. Something like a texture can be a vbm or a peg, so we need to keep track of which it is so we can substitute the correct gpu extension without having to do something like a string compare. That is what the split_extension_index value is. It stores to which set of extensions this file belongs.
 - primitive types are just statically assigned values
 - auxilary data is random data to attach to a container. We use that for zpp files I believe, but the streaming system just stores bytes and doesn't interpret it at all.
