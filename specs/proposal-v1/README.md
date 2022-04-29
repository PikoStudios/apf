# APF Proposal for v1

This proposal will include the following:
  - [x] Sectors
  - [x] Metadata sector (Header)
  - [x] Opcodes
  - [x] Checksum Sector
  - [x] Data sectors
  - [ ] Error Detection/Correction

## Sectors

All sectors start with sector starters. **Data sectors also require to be placed right in front of a Data Parity sector**

### Example
`0xFE [Metadata] 0xD1 [Data parity info] 0xD0 [Frame]`

### Plans

We plan in the future to add sector starters for more metadata about each sector, but for v1 we will only have sector footers

## Metadata sector (Header)

The metadata sector starts from `0x0` to whenever the sector footer is reached.

### Layout

`0x00 - 0x01` is reversed for version information.
 - `0x00` - Major version
 - `0x01` - Minor version

`0x02` is reversed for the **checksum** toggle which will be discussed about later. `0x02` can either be set to `0x00` or `0x01`

`0x03` is reversed for the **Target FPS**. Due to the `Target FPS` value only being one byte it can only go up to `0xFF` (255 FPS)

`0x04 - End of sector` is reversed for the address of the EOF opcode.

### Example

```
0x00 - 0x01
0x01 - 0x00
0x02 - 0x00
0x03 - 0x0F
0x04 - 0x05
0x05 - 0xEF
```

### Structure

```c
typedef struct apf_metadata
{
    unsigned char major_version;
    unsigned char minor_version;
    
    unsigned char fps;
    unsigned char use_checksum;
    
    uint64_t eof_opcode_address;
} apf_metadata;
```

## Opcodes

| Hexadecimal | Action |
| ------------- | ------------- |
| `0xEF` | EOF |
| `0xD0` | Start Data Sector |
| `0xD1` | Start Data Parity Sector |
| `0xDA` | Start Row Parity Sector |
| `0xDB` | Start Column Parity Sector |
| `0xDC` | Start Final Parity Sector |
| `0xCF` | Start Checksum Sector |

## Checksum Sector

The checksum sector should be placed after all of the data sectors, **but depending on the implemention, it could be placed anywhere after the metadata sector**

### Example

`0x01 0x00 [0x01] [..eof location..] 0xD1 [..parity information..] 0xD0 [..data sector..] 0xCF [..checksum..] 0xEF`

The `[0x01]` is the toggle to enable the **checksum** sector.

### Structure

```c
typedef uint32_t apf_checksum;
```

## Data Sector

The data sector is really made up of two sectors. the `Data Parity Sector` and the actual `Data Sector`. The `Data Parity Sector` goes before the `Data Sector ` and contains information for error detection. The `Data Sector` is made up of raw `PNG` bytes.

There is no way to specify which frame a `Data Sector` will be. The way frames are ordered is depending on the actual location relative to other data sectors. **For Example**, the first `Data Sector` will be the first frame and the third `Data Sector` will be the third frame.

### Example

`0xD1 [..parity information..] 0xD0 [..data sector..]`

### Structure

```c
typedef struct apf_data_sector
{
    struct parity
    {
        unsigned char** row_parity;
        unsigned char** column_parity;
        
        unsigned char final_parity;
    } parity;

    unsigned char data;
}
```
