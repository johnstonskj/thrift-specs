# Protocol API and Behavior

## Encoding-Agnostic Structure

[from Apache](https://github.com/apache/thrift/edit/master/doc/specs/thrift-protocol-spec.md)

```ebnf
message        = message-begin struct message-end
               ;
message-begin  = method-name message-type message-seqid
               ;
method-name    = STRING
               ;
message-type   = T_CALL | T_REPLY | T_EXCEPTION | T_ONEWAY
               ;
message-seqid  = I32
               ;
struct         = struct-begin field* field-stop struct-end
               ;
struct-begin   = struct-name
               ;
struct-name    = STRING
               ;
field-stop     = T_STOP
               ;
field          = field-begin field-data field-end
               ;
field-begin    = field-name field-type field-id
               ;
field-name     = STRING
               ;
field-type     = T_BOOL | T_BYTE | T_I8 | T_I16 | T_I32 | T_I64 | T_DOUBLE
               | T_STRING | T_BINARY | T_STRUCT | T_MAP | T_SET | T_LIST
               ;
field-id       = I16
               ;
field-data     = I8 | I16 | I32 | I64 | DOUBLE | STRING | BINARY
               | struct | map | list | set
               ;
map            = map-begin field-datum* map-end
               ;
map-begin      = map-key-type map-value-type map-size
               ;
map-key-type   = field-type
               ;
map-value-type = field-type
               ;
map-size       = I32
               ;
list           = list-begin field-data* list-end
               ;
list-begin     = list-elem-type list-size
               ;
list-elem-type = field-type
               ;
list-size      = I32
               ;
set            = set-begin field-data* set-end
               ;
set-begin      = set-elem-type set-size
               ;
set-elem-type  = field-type
               ;
set-size       = I32 ;
```

They key point to notice is that ALL messages are just one wrapped <struct>. Depending upon the message type, the <struct> can be interpreted as the argument list to a function, the return value of a function, or an exception.
  
As for unions,
  
```ebnf
union = struct-begin field field-stop struct-end ;
```


## Basic Types

T_*ID*     | ID | Type     | Comments
-----------|----|----------|-----------------------------------
`T_BOOL`   | 2  | `BOOL`   | Boolean value, `true` or `false`.
`T_BYTE`   | 3  | `BYTE`   | A single signed 8-bit byte.
`T_I8`     | 3  | `I8`     | A synonym for `T_BYTE`.
`T_I16`    | 6  | `I16`    | 16-bit signed integer.
`T_I32`    | 8  | `I32`    | 32-bit signed integer.
`T_I64`    | 10 | `I64`    | 64-bit signed integer.
`T_DOUBLE` | 4  | `DOUBLE` | IEEE 64-bit floating point.
`T_STRING` | 11 | `STRING` | Character string.
`T_BINARY` | 11 | `BINARY` | String of `T_BYTE`.

Character string may be UTF-7 or UTF-8.

### Additional Types

```thrift
enum AdditionlTypes {
  T_VOID = 1
  T_UTF8 = 16
  T_UTF16 = 17
}
```

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

[Protocol Template](https://johnstonskj.github.io/thrift-specs/protocol-template).
