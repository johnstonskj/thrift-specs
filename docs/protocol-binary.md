# Binary Protocol

This specification describes the binary wire encoding for Thrift, a required component of the minimal language support for Thrift.

This specification is based upon the [thrift-binary-protocol.md](https://github.com/apache/thrift/blob/master/doc/specs/thrift-binary-protocol.md) in GitHub which in turn states that it is based mostly on the *Java implementation in the Apache thrift library (version 0.9.1 and 0.9.3)*. 

## BNF For Binary Protocol

```ebnf
message        = message-header , struct ;
message-header = version-number , unused , message-type , name , sequence-id  (* strict *)
               | name , message-type , sequence-id (* older *)
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
size           = I32 ;
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

* `length` is the length of the byte array, a `T_I32` (must be >= 0).
* `bytes` are the `T_BYTE` values of the array.

## Message Encoding

A `message-header` can be encoded in two different ways, the first is a strict format with an encoded message version, and the second is an older form without version information.

```
Binary protocol Message, strict encoding, 12+ bytes:
+--------+--------+--------+--------+--------+--------+--------+--------+--------+...+--------+--------+--------+--------+--------+
|1vvvvvvv|vvvvvvvv|unused  |00000mmm| name length                       | name                | seq id                            |
+--------+--------+--------+--------+--------+--------+--------+--------+--------+...+--------+--------+--------+--------+--------+
```

```
Binary protocol Message, old encoding, 9+ bytes:
+--------+--------+--------+--------+--------+...+--------+--------+--------+--------+--------+--------+
| name length                       | name                |00000mmm| seq id                            |
+--------+--------+--------+--------+--------+...+--------+--------+--------+--------+--------+--------+
```

Where:

* `vvvvvvvvvvvvvvv` is the version, an unsigned 15 bit number fixed to `1` (in binary: `000 0000 0000 0001`).
  The leading bit is `1`.
* `unused` is an ignored `T_BYTE`.
* `mmm` is the message type, an unsigned 3 bit integer. 
  * The 5 leading bits must be `0` as some clients (checked for java in 0.9.1) take the whole byte.
* `name length` is the byte length of the name field, a `T_I32` (must be >= 0).
* `name` is the method name, a `T_STRING`.
* `seq id` is the sequence id, a `T_I32`.

Because `name length` **must be positive** (therefore the first bit is always `0`), the first bit allows the receiver to see
whether the strict format or the old format is used. Therefore a server and client using the different variants of the
binary protocol can transparently talk with each other. However, when strict mode is enforced, the old format is
rejected.

## Struct and Union Encoding

A *Struct* is a sequence of zero or more fields, followed by a stop field. Each field starts with a field header and
is followed by the encoded field value. 

Because each field header contains the field-id (as defined by the Thrift IDL file), the fields can be encoded in any
order. Thrift's type system is not extensible; you can only encode the primitive types and structs. Therefore is also
possible to handle unknown fields while decoding; these are simply ignored. While decoding the field type can be used to
determine how to decode the field value.

Note that the field name is not encoded so field renames in the IDL do not affect forward and backward compatibility.

The default Java implementation (Apache Thrift 0.9.1) has undefined behavior when it tries to decode a field that has
another field-type then what is expected. Theoretically this could be detected at the cost of some additional checking.
Other implementation may perform this check and then either ignore the field, or return a protocol exception.

A *Union* is encoded exactly the same as a struct with the additional restriction that at most 1 field may be encoded.

An *Exception* is encoded exactly the same as a struct.

### Field Encoding

Field headers and values are encoded in the following manner:

```
Binary protocol field header and field value:
+--------+--------+--------+--------+...+--------+
|tttttttt| field id        | field value         |
+--------+--------+--------+--------+...+--------+
```

Where:

* `tttttttt` the field-type, a `T_I8`.
* `field id` the field-id, a `T_I16`.
* `field-value` the encoded field value.

### Stop Field Handling

The stop field is simply encoded as the value `0` with the type `T_BYTE`.

```
Binary protocol stop field:
+--------+
|00000000|
+--------+
```

## List and Set Encoding

List and sets are encoded the same: a header indicating the size and the `element-type` of the elements, followed by the
encoded elements.

```
Binary protocol list (5+ bytes) and elements:
+--------+--------+--------+--------+--------+--------+...+--------+
|tttttttt| size                              | elements            |
+--------+--------+--------+--------+--------+--------+...+--------+
```

Where:

* `tttttttt` is the `element-type`, a `T_I8`
* `size` is the size, a `T_I32`, positive values only
* `elements` the element values

The maximum list/set size is configurable. By default there is no limit (meaning the limit is the maximum `I32` value:
2,147,483,647).

## Map Encoding

Maps are encoded with a header indicating the size, the element-type of the keys and the element-type of the elements,
followed by the encoded elements.

```
Binary protocol map (6+ bytes) and key value pairs:
+--------+--------+--------+--------+--------+--------+--------+...+--------+
|kkkkkkkk|vvvvvvvv| size                              | key value pairs     |
+--------+--------+--------+--------+--------+--------+--------+...+--------+
```

Where:

* `kkkkkkkk` is the `key-type`, a `T_I8`.
* `vvvvvvvv` is the value `element-type`, a `T_I8`.
* `size` is the size, a `T_I32`, positive values only
* `key value pairs` are the encoded keys and values

The maximum map size is configurable. By default there is no limit (meaning the limit is the maximum `I32` value:
2,147,483,647).

## Examples

TBD

## References

TBD

