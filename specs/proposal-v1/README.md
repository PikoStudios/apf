# APF Proposal for v1

This proposal will include the following:
  - [x] Sectors
  - [x] Metadata sector
  - [x] Opcodes
  - [ ] Confirming last action
  - [ ] Data sectors

## Sectors

All sectors end with a sector footer. The type of footer depends on the data.
**All section footerw must be confirmed by the confirm opcode**

### Example
`[Metadata] 0xFE 0xA6 [Data for the first frame] 0xD2 0xA6`

### Plans

We plan in the future to add sector starters for more metadata about each sector, but for v1 we will only have sector footers

## Metadata sector

The metadata sector starts from `0x0` to whenever the sector footer is reached.

### Layout

`0x00 - 0x01` is reversed for version information.
 - `0x00` - Major version
 - `0x01` - Minor version

`0x02` is reversed for the **checksum** toggle which will be discussed about later. `0x02` can either be set to `0x00` or `0x01`

`0x03 - Sector footer` is reversed for the address of the EOF opcode.

### Example

```
0x00 - 0x01
0x01 - 0x00
0x02 - 0x00
0x03 - 0x06
0x04 - 0xFE
0x05 - 0xA6
0x06 - 0xEF
0x07 - 0xA6
```

### Structure

```c
typedef struct apfMetadata
{
    unsigned char major_version;
    unsigned char minor_version;
    
    unsigned char checksum;
    unsigned char eof_opcode_address;
} apfMetadata;
```

## Opcodes

| Hexadecimal | Action |
| ------------- | ------------- |
| `0xA6` | Confirm last opcode  |
| `0xEF` | EOF |
| `0xFE` | End Metadata Sector |
| `0xD2` | End Data Sector |
| `0xCF` | End Checksum Sector |
