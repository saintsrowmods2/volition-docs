# Pegfiles
!!! note "File extensions"
    `.peg_pc`
    `.g_peg_pc` 


## Kaitai Struct

Based on [PDF documentation by SRM user Masamaru](https://www.saintsrowmods.com/forum/attachments/pegformat-pdf.9867/) ([Internet Archive backup](https://web.archive.org/web/20240220142652if_/https://www.saintsrowmods.com/forum/attachments/pegformat-pdf.9867/)).

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
    repeat-expr: header.entry_count
types:
  peg_header:
    seq:
      - id: magic
        contents: [0x47, 0x45, 0x4b, 0x56]
        doc: \'GEKV\' in ASCII
      - id: version
        type: u2
        doc: always 10 for SR2
      - id: unknown06
        type: u2
        doc: could be upper byte of version? seems to always be 0
      - id: file_size
        type: u4
        doc: file size of this file
      - id: data_file_size
        type: u4
        doc: file size of associated g_pec_pc file
      - id: entry_count
        type: u2
        doc: number of total textures. each texture can have one or more frames.
      - id: unknown12
        type: u2
      - id: frame_count
        type: u2
        doc: number of total frames.
      - id: unknown16
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
      - id: format
        type: u2
        enum: peg_format
      - id: unknown0a
        type: u2
      - id: unknown0c
        type: u4
      - id: frames
        type: u2
        doc: number of frame(s) in this entry
      - id: unknown12
        type: u4
      - id: unknown16
        type: u4
      - id: unknown_flags
        type: u1
        doc: Texture:0x01, Animation texture:0x1E
      - id: mipmap_count
        type: u1
        doc: mipmapcount (aka mipmaplevel)
      - id: size
        type: u4
        doc: size of image data
      - id: unknown20
        type: u4
      - id: unknown24
        type: u4
      - id: unknown28
        type: u4
      - id: unknown2c
        type: u4
enums:
  peg_format:
    400: dxt1
    401: dxt3
    402: dxt5
    403: r5g6b5
    404: a1r5g5b5
    405: a4r4g4b4
    406: r8g8b8
    407: a8r8g8b8
    408: v8u8
    409: cxv8u8
    410: a8
```