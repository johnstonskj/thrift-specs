# Binary Protocol

This specification describes the binary wire encoding for Thrift, a required component of the minimal language support for Thrift.

This specification is based upon the [thrift-binary-protocol.md](https://github.com/apache/thrift/blob/master/doc/specs/thrift-binary-protocol.md) in GitHub which in turn states that it is based mostly on the *Java implementation in the Apache thrift library (version 0.9.1 and 0.9.3)*. 

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

Type       | Format   | Comments
-----------|----------|---------
`T_BOOL`   | I8       | true becomes 1, false becomes 0.
`T_BYTE`   | I8       |
`T_I16`    | I16/NW   | Network (big-endian) order.
`T_I32`    | I32/NW   | Network (big-endian) order.
`T_I64`    | I64/NW   | Network (big-endian) order.
`T_DOUBLE` | I64/NW   | According to the IEEE 754 floating-point "double format" bit layout.
`T_STRING` | I32,{I8} | Encoded to UTF-8, and then treat as `T_BINARY`.
`T_BINARY` | I32,{I8} | Length, then bytes.

Note that all multi-byte **Integer** types are encoded in network (big-endian) order. Implementations may provide the option to use the binary protocol with little endian order (the standard C++ library does), but how this is negotiated is not current specified.

**Doubles** are converted to, and encoded as, `T_I64` according to the IEEE 754 floating-point "double format" bit layout.

**Strings** are first encoded to UTF-8, and then treated as binary.

**Binary** is encoded as follows:

```
Binary protocol, binary data, 4+ bytes:
+--------+--------+--------+--------+--------+...+--------+
| length                            | bytes               |
+--------+--------+--------+--------+--------+...+--------+
```

Where:

* `length` is the length of the byte array, a `T_I32` integer encoded in network (big endian) order (must be >= 0).
* `bytes` are the bytes of the byte array.

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

