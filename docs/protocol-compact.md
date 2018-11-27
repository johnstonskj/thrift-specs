# Compact Protocol 

## BNF For Compact Protocol

```ebnf
message               = version-and-type seq-id method-name struct-encoding
                      ;
version-and-type      = version (* 6-bit identifier *) 
                      , type    (* 2-bit identifier *)
                      ;
seq-id                = varint
                      ;
method-name           = string
                      ;
struct-encoding       = field_list , stop
                      ;
field_list            = field , field_list 
                      | field
                      ;
field                 = type-and-id , value
                      ;
type-and-id           = field-id-delta , type-header 
                      | "0" , type-header , zigzag-varint
                      ;
field-id-delta        = ? 4-bit offset from preceding field id, 1-15 ?
                      ;
type-header           = boolean-true | boolean-false | byte-type-header | i16-type-header
                      | i32-type-header | i64-type-header | double-type-header
                      | string-type-header | binary-type-header | list-type-header
                      | set-type-header | map-type-header | struct-type-header
                      ;
value                 = boolean-true | boolean-false | byte | i16 | i32 | i64 | double
                      | string | binary | list | set | map | struct
                      ;
stop                  = 0x0 ;
boolean-true          = 0x1 ;
boolean-false         = 0x2 ;
byte-type-header      = 0x3 ;
i16-type-header       = 0x4 ;
i32-type-header       = 0x5 ;
i64-type-header       = 0x6 ;
double-type-header    = 0x7 ;
binary-type-header    = 0x8 ;
string-type-header    = binary-type-header ;
list-type-header      = 0x9 ;
set-type-header       = 0xA ;
map-type-header       = 0xB ;
struct-type-header    = 0xC ;
byte                  = ? 1-byte value ? ;
i16                   = zigzag-varint ;
i32                   = zigzag-varint ;
i64                   = zigzag-varint ;
double                = ? 8-byte double ? ;
binary                = varint (* size *) { byte } ;
string                = binary (* utf-8 encoded *)
                      ;
list                  = type-header , varint , list-body
                      ;
set                   = type-header , varint , list-body
                      ;
list-body             = value , list-body 
                      | value
                      ;
map                   = type-header (* key *) , type-header (* value *) , varint (* size *)
                      , key-value-pair-list
                      ;
key-value-pair-list   = key-value-pair , key-value-pair-list 
                      | key-value-pair
                      ;
key-value-pair        = value (* key *) , value (* value *) ;
zigzag-varint         = (* see below *) ;
varint                = (* see below *) ;
```

## Basic Type Encoding

Type       | Format | Comments
-----------|--------|---------
`T_BOOL`   | TBD |
`T_BYTE`   | TBD |
`T_I16`    | TBD |
`T_I32`    | TBD |
`T_I64`    | TBD |
`T_DOUBLE` | TBD |
`T_STRING` | TBD |
`T_BINARY` | TBD |

## Message Encoding

TBD

## Struct and Union Encoding

TBD

### Field Encoding

TBD

Field Type     | Base Type  | ID | Comments
---------------|------------|--------------
`F_BOOL_TRUE`  | `T_BOOL`   | 1  |
`F_BOOL_FALSE` | `T_BOOL`   | 2  |
`F_BYTE`       | `T_BYTE`   | 3  |
`F_I16`        | `T_I16`    | 4  |
`F_I32`        | `T_I32`    | 5  |
`F_I64`        | `T_I64`    | 6  |
`F_DOUBLE`     | `T_DOUBLE` | 7  |
`F_BINARY`     | `T_BINARY` | 8  | Used for binary and string fields.
`F_LIST`       | `T_LIST`   | 9  |
`F_SET`        | `T_SET`    | 10 |
`F_MAP`        | `T_MAP`    | 11 |
`F_STRUCT`     | `T_STRUCT` | 12 | 

### Stop Field Handling

TBD

## List and Set Encoding

TBD

## Map Encoding

TBD

## Examples

TBD

## References

TBD

