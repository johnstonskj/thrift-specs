# Binary Protocol

## BNF For Binary Protocol

```ebnf
message        = [ version-number , unused , message-type ] (* strict *)
               , name , sequence-id , struct
               ;
version-number = I16 (* leading bit is 1, value is 1 *) ;
unused         = I8 ;
message-type   = I8 (* 5-bit leading must be 0, 3-bit value *) ;
name           = string ;
sequence-id    = I32 ;
struct         = { field-header , value } , stop-field ;
field-header   = field-type , field-id ;
field-type     = I8 ;
field-id       = I16 ;
value          = bool | I8 | I16 | I32 | I64 | DOUBLE
               | string | binary | struct | map | list-and-set ;
binary         = length , { BYTE } ;
length         = I32 ;
string         = binary (* utf-8 encoded *) ;
list-and-set   = element-type , size , { value } ;
element-type   = I8 ;
size           = I16 ;
map            = key-type , element-type , size , { key-value } ;
key-type       = I8 ;
key-value      = value (* key *) , value (* value *) ;
```

## Basic Type Encoding

Type       | Format  | Comments
-----------|---------|---------
`T_BOOL`   | I8      | true becomes 1, false becomes 0.
`T_BYTE`   | I8      |
`T_I16`    | I16/NW  | Network (big-endian) order.
`T_I32`    | I32/NW  | Network (big-endian) order.
`T_I64`    | I64/NW  | Network (big-endian) order.
`T_DOUBLE` | I64/NW  | According to the IEEE 754 floating-point "double format" bit layout.
`T_STRING` | I32+I8* | Encoded to UTF-8, and then treat as `T_BINARY`.
`T_BINARY` | I32+I8* | Length, then bytes.

## Message Encoding

TBD

## Struct and Union Encoding

TBD

### Field Encoding

TBD

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

