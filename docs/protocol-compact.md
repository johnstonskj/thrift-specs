# Compact Protocol 


This specification describes the *compact* wire encoding for Thrift, an optional component for Thrift and a required component for [Apache Parquet](https://parquet.apache.org/).

This specification is based upon the [thrift-compact-protocol.md](https://raw.githubusercontent.com/apache/thrift/master/doc/specs/thrift-compact-protocol.md) in GitHub which in turn states that it is based mostly on the Java implementation in the Apache thrift library (version 0.9.1) and
[THRIFT-110 A more compact format](https://issues.apache.org/jira/browse/THRIFT-110).

## BNF For Compact Protocol

The following is edited from the original in [THRIFT-110 A more compact format](https://issues.apache.org/jira/browse/THRIFT-110).

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
`T_BOOL`   | I8 or `type-header` | See [below](#Integer-Encoding-Details).
`T_BYTE`   | I8     |
`T_I16`    | zigzag-varint |
`T_I32`    | zigzag-varint |
`T_I64`    | zigzag-varint |
`T_DOUBLE` | zigzag-varint |
`T_STRING` | zigzag-varint,{I8} | Encoded to UTF-8, and then treat as `T_BINARY`.
`T_BINARY` | zigzag-varint,{I8} | Length, then bytes.

## Message Encoding

TBD

## Struct and Union Encoding

TBD

### Field Encoding

TBD

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

### Stop Field Handling

TBD

## List and Set Encoding

TBD

## Map Encoding

TBD

## Examples

TBD

## Integer Encoding Details

The *compact protocol* uses multiple encodings for integers: the _zigzag int_ (zigzag), and the _var int_ (varint).

### Zigzag Int Encoding

Values of type `T_I32` and `T_I64` are first transformed to a *zigzag int*. A zigzag int folds positive and negative
numbers into the positive number space. When we read 0, 1, 2, 3, 4 or 5 from the wire, this is translated to 0, -1, 1,
-2 or 2 respectively. Here are the (Scala) formulas to convert from int32/int64 to a zigzag int and back:

```scala
def intToZigZag(n: Int): Int = (n << 1) ^ (n >> 31)
def zigzagToInt(n: Int): Int = (n >>> 1) ^ - (n & 1)
def longToZigZag(n: Long): Long = (n << 1) ^ (n >> 63)
def zigzagToLong(n: Long): Long = (n >>> 1) ^ - (n & 1)
```

To encode a `T_I16` as zigzag int, it is first converted to an `T_I32` and then encoded as such. The type `T_I8` simply
uses a single byte as in the binary protocol.

### Var Int Encoding

The zigzag int is then encoded as a *var int*. Var ints take 1 to 5 bytes (int32) or 1 to 10 bytes (int64). The most
significant bit of each byte indicates if more bytes follow. The concatenation of the least significant 7 bits from each
byte form the number, where the first byte has the most significant bits (so they are in big endian or network order).

Var ints are sometimes used directly inside the compact protocol to represent positive numbers.


