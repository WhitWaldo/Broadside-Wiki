# Forge File Format

The file format used by Assassin Creed IV's AnvilNext engine (previously named Anvil and Scimitar) to store resources has the `.forge` filename extension. In AC4, these files all fit the `DataPC_*\.forge` regex. This article attempts to shed light on the workings of this file format.

## Data Description

This is the breakdown of the format. Type are as listed below:
```
(u|i)(8,16,32,64)(LE,BE,)
  ^       ^         ^^
  |       |          `- Little Endian or Big Endian (default is Little Endian) 
  |       `------------ The size of the type, in bits
  `-------------------- The signedness of the type. 'u' is for unsigned, 'i' is for signed

Special types:
char   : u8;
wchar  : u16;
```

Special values:
```c
$  : address of current data
$$ : address of current chunk
$? : address of current table/list item
?? : current index in list
```

### The File Header Chunk

```C
0x00000000: file_hdr:
file_hdr+0x00000000: .scimitar:        char[8] = 'scimitar'   // File identifier
file_hdr+0x00000008: .null_byte:       char    = 0x00         // Possibly null terminator?
file_hdr+0x00000009: .unknown:         u32                    // Possibly file format version?
file_hdr+0x0000000d: .rsc_idx_hdr_ptr: u32                    // Pointer to Resource Indexes Header
file_hdr+0x00000011: .unknown:         u32[4]
file_hdr+0x00000021: .padding:         char[rsc_idx_hdr_ptr-$]
file_hdr_end:
```

### The Resource Index Chunk


#### The Resource Index Header
```c
file_hdr.rsc_idx_hdr_ptr: rsc_idx_hdr:
rsc_idx_hdr+0x00000000: .record_count:     u32     // Number of resources in this file
rsc_idx_hdr+0x00000004: .unknown:          u32[10]
rsc_idx_hdr+0x0000002b: .unknown:          u32     // Seems to be the same as .record_count
rsc_idx_hdr+0x00000030: .unknown:          u32[7]
rsc_idx_hdr+0x0000004c: .rsc_desc_ptr:     u32     // Pointer to the Resource Descriptions Chunk
rsc_idx_hdr+0x00000050: .unknown:          u32
rsc_idx_hdr+0x00000054: .useless_chk_ptr:  u32     // Pointer to the Useless Chunk
rsc_idx_hdr+0x00000058: .unknown:          u32
rsc_idx_hdr+0x0000006c: .rsc_data_chk_ptr: u32     // Pointer to the Resource Data Section
rsc_idx_hdr_end:
```

#### The Resource Index List
Located at `file_hdr.rsc_idx_hdr_ptr`.
A list that is `rsc_idx_hdr.record_count` elements large, containing:
```c
resource_record_??:
resource_record_??+0x00000000: .resource_ptr: u32     // A pointer to the resource data
resource_record_??+0x00000004: .unknown:       u32[4]
resource_record_??_end
```

### Resource Descriptions Chunk
Located at `rsc_idx_hdr.rsc_desc_ptr`.
A list that is `rsc_idx_hdr.record_count` elements large, containing:
```c
resource_desc_??:
resource_desc_??+0x00000000: .resource_size: u32 
resource_desc_??+0x00000004: .unknown:       u32[6]
resource_desc_??+0x0000001c: .next_index:    u32 = ?? + 1 // or ~0 if ?? is rsc_idx_hdr.record_count-1
resource_desc_??+0x00000020: .prev_index:    u32 = ?? - 1 // or ~0 if ?? is 0
resource_desc_??+0x00000024: .unknown:       u32
resource_desc_??+0x00000028: .timestamp:     u32          // UNIX timestamp
resource_desc_??+0x0000002c: .name:          char[128]    // The name of the file, padded with 0x0
resource_desc_??+0x00000078: .unknown:       u32[5]
```

### Resource Data Chunk
This chunk starts at `file_hdr..rsc_data_chk_ptr`. It contains some unknown values and then a UUID in ASCII.
The rest of the data in this chunk corresponds to each `resource_record_??.resource_ptr`. That pointer leads to the following sequence of bytes every time, but in a different location: 
```
04 00 00 00 : 0x4
00 00 00 00 : 0x0
33 AA FB 57
99 FA 04 10
01 00 00 00 : 0x1
80 00 80 01
```

The sequence is then repeated several bytes (at least 20 bytes) later, except without the first 32-bit `0x4`.