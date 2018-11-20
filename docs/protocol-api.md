# Protocol API and Behavior


```
interface TProtocol {
  writeMessageBegin(name : string, type : call-type, seq : natural)
  writeMessageEnd()
  writeStructBegin(name : string)
  writeStructEnd()
  writeFieldBegin(name : string, type data-type, id : natural)
  writeFieldEnd()
  writeFieldStop()
  writeMapBegin(ktype : data-type, vtype : data-type, size : positive)
  writeMapEnd()
  writeListBegin(etype : data-type, size : positive)
  writeListEnd()
  writeSetBegin(etype : data-type, size : positive)
  writeSetEnd()
  writeBool(v : bool)
  writeByte(v : byte)
  writeI16(v : i16)
  writeI32(v : i32)
  writeI64(v : i64)
  writeDouble(v : double)
  writeString(v : string)

  readMessageBegin() : (name : string, type : data-type, seq : natural)
  readMessageEnd()
  readStructBegin() : string
  readStructEnd()
  readFieldBegin() : (name : string, type : data-type, id : natural)
  readFieldEnd()
  readMapBegin() : (k : data-type, v : data-type, size : positive)
  readMapEnd()
  readListBegin() : (etype : data-type, size : positive)
  readListEnd()
  readSetBegin() : (etype : : data-type, size : positive)
  readSetEnd()
  readBool() : bool
  readByte() : byte
  readI16() : i16
  readI32() : i32
  readI64() : i64
  readDouble() : double
  readString() : string
}
```

```
enum call-type {
  call = 1
  reply = 2
  exception = 3
  one-way = 4
}
```

```
enum data-type {
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
const stop-field-value = 0
```

```
enum more-data-types {
  void = 1
  utf-8 = 16
  utf-16 = 17
}
```
