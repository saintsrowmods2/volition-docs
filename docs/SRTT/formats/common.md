# Common

## v_file_header
```cpp
struct v_file_header
{
    et_uint16        m_signature;   // binary file signature
    et_uint16        m_version;     // file version

    et_uint32        m_reference_data_size;     // size of reference data
    et_uint32        m_reference_data_start;    // offset to reference data
    et_uint32        m_reference_count;         // number of references in header

    uint8            m_initialized; // whether this header has been initialized.
    uint8            m_pad[15];     // extra pad data, zero'd before write
}; 
```