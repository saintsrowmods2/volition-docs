# g_chunk_pc

The GPU chunk file only contains vertex and index buffers. The file consists of two major parts:
  
## Shared buffers section
Massive shared vertex and index buffers for static world geometry.

Contents:

- 0..n vertex buffers for each type of vertex. Type means the number of UV's and such in a vertex. 
- A single index buffer for the above vertex buffers.

## Individual models section

- The CPU chunk header contains a pointer to this section.
- The contents is obviously a bunch of models, but where exactly are the headers is not fully figured out.
- The shared buffer seems to return garbage for all "loose" and destroyable objects, so it's probably safe to assume this section is the one holding them instead.