# Protocol API and Behavior

```Thrift
interface TProtocol {
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

```
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

```
enum CallType {
  call = 1
  reply = 2
  exception = 3
  one-way = 4
}
```

```
enum Type {
  stop = 0
  bool = 2
  byte = 3
  double = 4
  int16 = 6
  int32 = 8
  int64 = 10
  string = 11
  struct = 12
  map = 13
  set = 14
  list = 15
}
```

```
const STOP_VALUE = 0
```

```
enum AdditionlTypes {
  void = 1
  utf-8 = 16
  utf-16 = 17
}
```

