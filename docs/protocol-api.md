# Protocol API and Behavior

## Encoding-Agnostic Structure

[from Apache](https://github.com/apache/thrift/edit/master/doc/specs/thrift-protocol-spec.md)

```ebnf
message        = message-begin , struct , message-end
               ;
message-begin  = method-name , message-type , message-seqid
               ;
method-name    = string
               ;
message-type   = T_CALL | T_REPLY | T_EXCEPTION | T_ONEWAY
               ;
message-seqid  = i32
               ;
struct         = struct-begin , { field } , field-stop , struct-end
               ;
struct-begin   = struct-name
               ;
struct-name    = string
               ;
field-stop     = T_STOP
               ;
field          = field-begin , field-data , field-end
               ;
field-begin    = field-name , field-type , field-id
               ;
field-name     = string
               ;
field-type     = T_BOOL | T_BYTE | T_I8 | T_I16 | T_I32 | T_I64 | T_DOUBLE
               | T_STRING | T_BINARY | T_STRUCT | T_MAP | T_SET | T_LIST
               ;
field-id       = i16
               ;
field-data     = i8 | i16 | i32 | i64 | double | string | binary
               | struct | map | list | set
               ;
map            = map-begin , { field-datum } , map-end
               ;
map-begin      = map-key-type , map-value-type , map-size
               ;
map-key-type   = field-type
               ;
map-value-type = field-type
               ;
map-size       = i32
               ;
list           = list-begin , { field-data } , list-end
               ;
list-begin     = list-elem-type , list-size
               ;
list-elem-type = field-type
               ;
list-size      = i32
               ;
set            = set-begin , { field-data } , set-end
               ;
set-begin      = set-elem-type , set-size
               ;
set-elem-type  = field-type
               ;
set-size       = i32 ;
```

They key point to notice is that ALL messages are just one wrapped <struct>. Depending upon the message type, the <struct> can be interpreted as the argument list to a function, the return value of a function, or an exception.

## Basic Types

T_*ID*     | ID | IDL Type | Comments
-----------|----|----------|-----------------------------------
`T_BOOL`   | 2  | `bool`   | Boolean value, `true` or `false`.
`T_BYTE`   | 3  | `byte`   | A single signed 8-bit byte.
`T_I8`     | 3  | `i8`     | A synonym for `T_BYTE`.
`T_I16`    | 6  | `i16`    | 16-bit signed integer.
`T_I32`    | 8  | `i32`    | 32-bit signed integer.
`T_I64`    | 10 | `i64`    | 64-bit signed integer.
`T_DOUBLE` | 4  | `double` | IEEE 64-bit floating point.
`T_STRING` | 11 | `string` | Character string.
`T_BINARY` | 11 | `binary` | String of `T_BYTE`.

Notes:

* Character string may be UTF-7 or UTF-8.
* Unless otherwise specified in a protocol, enumeration values are encoded as `T_I32` values.

## Structured Types

T_*ID*     | ID | Comments
-----------|----|-----------------------------------
`T_STRUCT` | 12 |
`T_MAP`    | 13 |
`T_SET`    | 14 |
`T_LIST`   | 15 |

Unless otherwise specified in a protocol, a **union** is encoded in the same way as a structure, except that it enforces the rule that one, and only one, field may be present at any time, as in:
  
```ebnf
union = struct-begin field field-stop struct-end ;
```

Unless otherwise specified in a protocol, an **exception** is encoded in the same way as a structure.

<!--
## Additional Types (From PyThrift2)

```thrift
enum AdditionlTypes {
  T_VOID = 1
  T_UTF8 = 16
  T_UTF16 = 17
}
```
-->

## Stop Field

```thrift
const i8 T_STOP = 0
```

## Call Types

```thrift
enum CallType {
  T_CALL = 1
  T_REPLY = 2
  T_EXCEPTION = 3
  T_ONEWAY = 4
}
```

## Protocol Interface

```thrift
/* interface */ service TProtocol {
  void writeMessageBegin(1: MessageHeader message),
  void writeMessageEnd(),
  void writeStructBegin(1: string name),
  void writeStructEnd(),
  void writeFieldBegin(1: FielHeader field),
  void writeFieldEnd(),
  void writeFieldStop(),
  void writeMapBegin(1: MapHeader map),
  void writeMapEnd(),
  void writeListBegin(1: ListHeader list)
  void writeListEnd(),
  void writeSetBegin(1: ListHeader set)
  void writeSetEnd(),
  void writeBool(1: bool v)
  void writeByte(1: byte v)
  void writeI16(1: i16 v)
  void writeI32(1: i32 v)
  void writeI64(1: i64 v)
  void writeDouble(1: double v)
  void writeString(1: string v)

  MessageHeader readMessageBegin(),
  void readMessageEnd(),
  string readStructBegin(),
  void readStructEnd(),
  FieldHeader readFieldBegin(),
  void readFieldEnd(),
  MapHeader readMapBegin(),
  void readMapEnd(),
  ListHeader readListBegin(),
  void readListEnd(),
  ListHeader readSetBegin(),
  void readSetEnd(),
  bool readBool(),
  byte readByte(),
  i16 readI16(),
  i32 readI32(),
  i64 readI64(),
  double readDouble(),
  string readString()
}
```

### Interface Types

```thrift
struct MessageHeader {
  1: string name,
  2: Calltype type,
  3: i32 sequence
}

struct FieldHeader {
  1: string name,
  2: dataType Type,
  3: i32 id
}

struct ListHeader {
  1: Type elementType,
  2: i32 size
}

struct MapHeader {
  1: Type keyType,
  2: Type elementType,
  3: i32 size
}
```

### Exceptions

When the message is of type Exception the struct is encoded as if it was declared by the following IDL:

```thrift
exception TApplicationException {
  1: string message,
  2: i32 type
}
```

The following exception types are defined in the Java implementation (0.9.3):

Code | Type  | Usage in Java Implementation
-----|-------|--------
0    | unknown              | Used in case the type from the peer is unknown.
1    | unknown method       | Used in case the method requested by the client is unknown by the server.
2    | invalid message type | None found.
3    | wrong method name    | None found.
4    | bad sequence id      | Used internally by the client to indicate a wrong sequence id in the response.
5    | missing result       | Used internally by the client to indicate a response without any field (result nor exception).
6    | internal error       | Used when the server throws an exception that is not declared in the Thrift IDL file. 
7    | protocol error       | Used when something goes wrong during decoding. For example when a list is too long or a required field is missing. 
8    | invalid transform    | None found.
9    | invalid protocol     | None found.
10   | unsupported client type | None found.

[Protocol Template](https://johnstonskj.github.io/thrift-specs/protocol-template).
