# Pegfiles
!!! note "File extensions"
    `.peg_pc`
    `.g_peg_pc` 


## Kaitai Struct

Based on [PDF documentation by SRM user Masamaru](https://www.saintsrowmods.com/forum/attachments/pegformat-pdf.9867/) ([Internet Archive backup](https://web.archive.org/web/20240220142652if_/https://www.saintsrowmods.com/forum/attachments/pegformat-pdf.9867/)). Further details taken from [SRM post by Knobbly](https://www.saintsrowmods.com/forum/threads/retexture-help.2630/) ([Internet Archive backup](https://web.archive.org/web/20240222224545/https://www.saintsrowmods.com/forum/threads/retexture-help.2630/)).

```
meta:
  id: saints_row_2_peg_pc
  file-extension: peg_pc
  endian: le
instances:
  header:
    type: peg_header
    pos: 0x0
    size: 0x18
  frame_data:
    type: peg_frame
    pos: 0x18
    repeat: expr
    repeat-expr: header.frame_count
  entry_names:
    pos: 24 + (48 * header.frame_count)
    type: str
    encoding: ascii
    terminator: 0
    repeat: expr
    repeat-expr: header.bitmap_count
types:
  peg_header:
    seq:
      - id: magic
        contents: [0x47, 0x45, 0x4b, 0x56]
      - id: version
        type: u2
        doc: 10 for SR2; 13 in SRTT
      - id: platform
        type: u2
      - id: directory_block_size
        type: u4
        doc: file size of this file
      - id: data_block_size
        type: u4
        doc: file size of associated g_pec_pc file
      - id: bitmap_count
        type: u2
        doc: number of total textures. each texture can have one or more frames.
      - id: flags
        type: u2
      - id: frame_count
        type: u2
        doc: number of total frames.
      - id: alignment_value
        type: u2
  peg_frame:
    seq:
      - id: offset
        type: u4
        doc: offset to the image data in the g_peg_pc file. offset value must be multiple of 16.
      - id: width
        type: u2
        doc: image width
      - id: height
        type: u2
        doc: image height
      - id: bitmap_format
        type: u2
        enum: bitmap_format
      - id: pal_format
        type: u2
      - id: mip_filter_value
        type: u4
      - id: frame_count
        type: u2
        doc: number of frame(s) in this entry
      - id: flags
        size: 2
        type: peg_entry_flags
      - id: filename_offset
        type: u4
      - id: pal_size
        type: u2
      - id: fps
        type: u1
      - id: mip_levels
        type: u1
        doc: base frame + number of mips, always at least 1
      - id: frame_size
        type: u4
        doc: bytes in palette + image
      - id: next_pointer
        type: u4
        doc: runtime
      - id: prev_pointer
        type: u4
        doc: runtime
      - id: cache1
        type: u4
        doc: runtime
      - id: cache2
        type: u4
        doc: runtime
  peg_entry_flags:
    seq:
      - id: alpha
        type: b1
      - id: non_power_of_two
        type: b1
      - id: alpha_test
        type: b1
      - id: cube_map
        type: b1
      - id: interleaved_mips
        type: b1
      - id: interleaved_data
        type: b1
      - type: b1
      - type: b1
enums:
  bitmap_format:
    400: dxt1
    401: dxt3
    402: dxt5
    403: r5g6b5
    404: a1r5g5b5
    405: a4r4g4b4
    406: r8g8b8
    407: a8r8g8b8
    408: v8u8 # aka 16_DUDV
    409: cxv8u8 # aka 16_DOT3_COMPRESSED 
    410: a8
```