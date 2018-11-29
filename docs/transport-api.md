# Transport API and Behavior

This specification describes the generic transport layer interface and behavior, not any one specific implementation. The transport layer provides an interface that supports the reading and writing of bytes only, no encoding or decoding which is the responsibility of the [protocol layer](https://johnstonskj.github.io/thrift-specs/protocol-api).

## Client Transport API

```thrift
(* interface *) service TTransport {
  void open(),
  void close(),
  bool isOpen(),
  bool peek(),
  binary read(1 : i16 bytes),
  write(1 : binary bytes),
  void flush()
}
```

We assume constructor or factory methods to create instances of these.

```thrift
(* interface *) service TBufferedTransport {
  i32 maxSize(),
  binary getBuffer(),
  i32 position(),
  i32 remaining(),
  void consume(1: i32 bytes)
}
```

```thrift
(* interface *) service TSeekableTransport {
  i32 length(),
  i32 position(),
  void setPosition(1: i32 position)
}
```

## Server Transport API

```thrift
(* interface *) service TServerTransport {
  void open(),
  void listen(),
  TTransport accept(),
  void close()
}
```
