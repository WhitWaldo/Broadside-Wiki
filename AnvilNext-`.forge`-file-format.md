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

### The File Header Chunk

```C
0x00000000: char  scimitar[8] = 'scimitar' // File identifier
0x00000008: char  null_byte   = 0x00       // Possibly null terminator?
0x00000009: u32   unknown                  // Possibly file format version?
```