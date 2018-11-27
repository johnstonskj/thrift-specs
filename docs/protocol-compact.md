# Compact Protocol 


This specification describes the *compact* wire encoding for Thrift, an optional component for Thrift and a required component for [Apache Parquet](https://parquet.apache.org/). The intent of the compact protocol is to be more space efficient than the existing [binary protocol](https://johnstonskj.github.io/thrift-specs/protocol-binary).

This specification is based upon the [thrift-compact-protocol.md](https://raw.githubusercontent.com/apache/thrift/master/doc/specs/thrift-compact-protocol.md) in GitHub which in turn states that it is based mostly on the Java implementation in the Apache thrift library (version 0.9.1) and
[THRIFT-110 A more compact format](https://issues.apache.org/jira/browse/THRIFT-110).

This specification refers to the [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-api) which defines protocol-agnostic and default behavior.

## BNF For Compact Protocol

The following is edited from the original in [THRIFT-110 A more compact format](https://issues.apache.org/jira/browse/THRIFT-110).

```ebnf
message               = message-header ,  struct-encoding
                      ;
message-header        = version-and-type , seq-id , method-name
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
map                   = empty-map | non-empty-map ;
empty-map             = ? size = 0 ? ;
non-empty-map         = varint (* size *)
                      , type-header (* key: 4 bits *) , type-header (* element: 4 bits *) 
                      , key-value-pair-list
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
`T_BOOL`   | I8 or `type-header` | See [below](#Integer-Encoding-Details).
`T_BYTE`   | I8     |
`T_I16`    | zigzag-varint |
`T_I32`    | zigzag-varint |
`T_I64`    | zigzag-varint |
`T_DOUBLE` | zigzag-varint |
`T_STRING` | varint,{I8} | Encoded to UTF-8, and then treat as `T_BINARY`.
`T_BINARY` | varint,{I8} | Length, then bytes.

Note that all multi-byte **Integer** types are encoded in network (big-endian) order before any zigzag/varint encoding. Implementations may provide the option to use the binary protocol with little endian order (the standard C++ library does), but how this is negotiated is not current specified.

**Doubles** are converted to, and encoded as, `T_I64` (zigzag-varint) according to the IEEE 754 floating-point "double format" bit layout.

**Strings** are first encoded to UTF-8, and then treated as binary.

**Binary** is encoded as follows:

```
Binary protocol, binary data, 4+ bytes:
+--------+--------+--------+--------+--------+...+--------+
| length                            | bytes               |
+--------+--------+--------+--------+--------+...+--------+
```

Where:

* `length` is the length of the byte array, encoded as a `T_I32` varint (must be >= 0).
* `bytes` are the `T_BYTE` values of the array.

## Message Encoding

A `message-header` is encoded as follows:

```
Compact protocol message-header (4+ bytes):
+--------+--------+--------+...+--------+--------+...+--------+--------+...+--------+
|pppppppp|mmmvvvvv| seq id              | name length         | name                |
+--------+--------+--------+...+--------+--------+...+--------+--------+...+--------+
```

Where:

* `pppppppp` is the protocol id, fixed to `1000 0010`, `0x82`.
* `mmm` is the message type, an unsigned 3 bit integer.
* `vvvvv` is the version, an unsigned 5 bit integer, fixed to `00001`.
* `seq id` is the sequence id, a `T_I32` encoded as a varint.
* `name length` is the byte length of the name field, a `T_I32` encoded as a varint (must be >= 0).
* `name` is the method name to invoke, a UTF-8 encoded string.

## Struct and Union Encoding

A *struct* is a sequence of zero or more fields, followed by a stop field. Each field starts with a field header and
is followed by the encoded field value. 

Because each field header contains the field-id (as defined by the Thrift IDL file), the fields can be encoded in any
order. Thrift's type system is not extensible; you can only encode the primitive types and structs. Therefore is also
possible to handle unknown fields while decoding; these are simply ignored. While decoding the field type can be used to
determine how to decode the field value.

Note that the field name is not encoded so field renames in the IDL do not affect forward and backward compatibility.

The default Java implementation (Apache Thrift 0.9.1) has undefined behavior when it tries to decode a field that has
another field-type than what is expected. Theoretically this could be detected at the cost of some additional checking.
Other implementation may perform this check and then either ignore the field, or return a protocol exception.

### Field Encoding

```
Compact protocol field header (short form) and field value:
+--------+--------+...+--------+
|ddddtttt| field value         |
+--------+--------+...+--------+

Compact protocol field header (1 to 3 bytes, long form) and field value:
+--------+--------+...+--------+--------+...+--------+
|0000tttt| field id            | field value         |
+--------+--------+...+--------+--------+...+--------+
```

Where:

* `dddd` is the field id delta, an unsigned 4 bits integer, strictly positive.
* `tttt` is field-type id, an unsigned 4 bit integer.
* `field id` the field id, a `T_I16` encoded as zigzag int.
* `field-value` the encoded field value.

The field id delta can be computed by `current-field-id - previous-field-id`, or just `current-field-id` if this is the
first of the struct. The short form should be used when the field id delta is in the range `1 - 15` (inclusive).

The following field-types can be encoded:

Field Type     | Base Type  | ID | Comments
---------------|------------|----|---------
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

Note that because there are 2 specific field types for the boolean values, the encoding of a boolean field value has no
length (0 bytes).

### Stop Field Handling

The stop field is simply encoded as the value `0` with the type `T_I8`.

```
Binary protocol stop field:
+--------+
|00000000|
+--------+
```


## List and Set Encoding

List and sets are encoded the same: a header indicating the size and the element-type of the elements, followed by the
encoded elements.

```
Compact protocol list header (1 byte, short form) and elements:
+--------+--------+...+--------+
|sssstttt| elements            |
+--------+--------+...+--------+

Compact protocol list header (2+ bytes, long form) and elements:
+--------+--------+...+--------+--------+...+--------+
|1111tttt| size                | elements            |
+--------+--------+...+--------+--------+...+--------+
```

Where:

* `ssss` is the size, 4 bit unsigned int, values `0` - `14`
* `tttt` is the element-type, a 4 bit unsigned int
* `size` is the size, a `T_I32` varint, positive values `15` or higher
* `elements` are the encoded elements

The short form should be used when the length is in the range `0 - 14` (inclusive).

The maximum list/set size is configurable. By default there is no limit (meaning the limit is the maximum `T_I32` value:
2,147,483,647).

## Map Encoding

Maps are encoded with a header indicating the size, the type of the keys and the element-type of the elements, followed
by the encoded elements. As the size is the first encoded element a value of `0` indicates an empty map and no further encoded data.

```
Compact protocol map header (1 byte, empty map):
+--------+
|00000000|
+--------+

Compact protocol map header (2+ bytes, non empty map) and key value pairs:
+--------+...+--------+--------+--------+...+--------+
| size                |kkkkvvvv| key value pairs     |
+--------+...+--------+--------+--------+...+--------+
```

Where:

* `size` is the size, a `T_I32` varint, strictly positive values
* `kkkk` is the key element-type, a 4 bit unsigned int
* `vvvv` is the value element-type, a 4 bit unsigned int
* `key value pairs` are the encoded keys and values

The element-types are the same as for lists. The full list is included in the 'List and set' section.

The maximum map size is configurable. By default there is no limit (meaning the limit is the maximum * `size` is the size, a `T_I32`varint (int32), strictly positive values value: 2,147,483,647).

## Examples

TBD

## Integer Encoding Details

The *compact protocol* uses multiple encodings for integers: the _zigzag int_ (zigzag), and the _var int_ (varint).

### Zigzag Int Encoding

Values of type `T_I32` and `T_I64` are first transformed to a *zigzag int*. A zigzag int folds positive and negative
numbers into the positive number space. When we read 0, 1, 2, 3, 4 or 5 from the wire, this is translated to 0, -1, 1,
-2 or 2 respectively. To encode a `T_I16` as zigzag int, it is first converted to an `T_I32` and then encoded as such. The type `T_I8` simply uses a single byte as in the binary protocol.

Here are the Scala formulas to convert from `T_I32`/`T_I64` to a zigzag int and back:

```scala
def intToZigZag(n: Int): Int = (n << 1) ^ (n >> 31)
def zigzagToInt(n: Int): Int = (n >>> 1) ^ - (n & 1)
def longToZigZag(n: Long): Long = (n << 1) ^ (n >> 63)
def zigzagToLong(n: Long): Long = (n >>> 1) ^ - (n & 1)
```

Here is the Python implementation:

```python
def make_zig_zag(n, bits):
    return (n << 1) ^ (n >> (bits - 1))

def from_zig_zag(n):
    return (n >> 1) ^ -(n & 1)
```

Finally, a Racket (Scheme) implementation:

```racket
(define (integer->zigzag n)
  (cond
    [(< (integer-length n) 32)
     (bitwise-xor (arithmetic-shift n 1) (arithmetic-shift n -31))]
    [(< (integer-length n) 64)
     (bitwise-xor (arithmetic-shift n 1) (arithmetic-shift n -63))]
    [else (raise (number-too-large (current-continuation-marks) 64 n))]))

(define (zigzag->integer z)
  (bitwise-xor (arithmetic-shift z -1) (- (bitwise-and z 1))))
```

### Varint Encoding

The zigzag int is then encoded as a *variable-length integer*. Varints take 1 to 5 bytes (`T_I32`) or 1 to 10 bytes (`T_I64`). The most significant bit of each byte indicates if more bytes follow. The concatenation of the least significant 7 bits from each byte form the number, where the first byte has the most significant bits (so they are in big endian or network order).

Varints are sometimes used directly inside the compact protocol to represent positive numbers.

Here is the Python implementation:

```python
def write_varint(trans, n):
    out = []
    while True:
        if n & ~0x7f == 0:
            out.append(n)
            break
        else:
            out.append((n & 0xff) | 0x80)
            n = n >> 7
    data = array.array('B', out).tostring()
    trans.write(data)

def read_varint(trans):
    result = 0
    shift = 0
    while True:
        x = trans.read(1)
        byte = ord(x)
        result |= (byte & 0x7f) << shift
        if byte >> 7 == 0:
            return result
        shift += 7
```
