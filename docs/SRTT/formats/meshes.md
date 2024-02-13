# Meshes
!!! note inline end "File extensions"
    `.ccmesh_pc`
    `.csmesh_pc`
    `.gsmesh_pc`
    `.clmesh_pc`
    `.glmesh_pc`
    `.rig_pc`
??? note "Links"
    - https://www.saintsrowmods.com/forum/threads/peg-file-format.2908/
??? note "Tools"
    -
___

Look at [SRIV equivalent,](../../SRIV/formats/meshes) it's probably similar.



## .ccmesh_pc
Characters


## .clmesh_pc
Static world models  
Seem to contain havok collisions

??? note "KSY"

    ```yaml

    meta:
    id: srtt_clmesh_pc
    file-extension: clmesh_pc
    endian: le
    encoding: utf-8
    doc: |
        SRTT Level Mesh Header
        Very, very WIP

    seq:
    - id: file_header
        type: v_file_header
        
    - id: texture_names
        type: strz
        repeat: expr
        repeat-expr: file_header.reference_count
    - size: 1
    - type: align(16)
    
    - id: level_mesh_headerr
        type: level_mesh_header
    #- id: unk1
    #  type: u4
    #- id: unk2
    #  type: u4
        #contents: [20,0,0,0]
    #- id: unk3
    #  type: u4
    - id: unk4
        type: f4
        repeat: expr
        repeat-expr: 15
    - id: unk5
        type: s4
    - contents: [-1,-1,-1,-1]
    - size: 64
    - id: unk6
        type: f4
        repeat: expr
        repeat-expr: 6
    - size: 32
    - id: unk7
        type: u4
    - id: unk8
        type: u4
    - size: 96
    - id: unk9
        type: u4
    - id: unk10
        type: u4
    
    types:
    
    align:
        doc: |
        Byte alignment tool
        params:
        - id: size
            type: u4
        seq:
        - size: (size - _io.pos) % size
    
    v_file_header:
        doc: |
        Standard Volition file header
        seq:
        - id: signature
            type: u2
        - id: version
            type: u2
        - id: reference_data_size
            type: u4
        - id: reference_data_start
            type: u4
        - id: reference_count
            type: u4
        - id: initialized
            type: u1
        - size: 15
        
    fl_vector:
        seq:
        - id: x
            type: f4
        - id: y
            type: f4
        - id: z
            type: f4
    
    bbox:
        seq:
        - id: min
            type: fl_vector
        - id: max
            type: fl_vector
    
    level_mesh_header:
        doc: |
        This is a guess based on SRIV docs
        seq:
        - id: signature
            type: u4
        - id: version
            type: u2
        - size: 2
        - id: mesh_flags
            doc: probably
            type: u4
        - id: bounding_center
            type: fl_vector
        - id: bounding_radius
            type: f4
        - id: unk2
            type: f4
        - id: unk3
            type: f4
        - id: local_aabb
            type: bbox
    ```
        
        
        
        


## .glmesh_pc
Static world models