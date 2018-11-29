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

The behavior of a transport method other than `open` being called when the transport is not yet opened is implementation specific.

> Note that the chosen name `peek` can be problematic in some implementations where this is the language/library name of an procedure to retrieve the next byte/bytes from a stream without moving the position within the stream. In such cases the Thrift `peek` may be renamed according to language-specific idioms.

We assume that distinct constructor, or factory, procedures exist in a transport implementation to create instances of specific transports.

### Additional Capability Interfaces

The following set of methods (taken from the Java implementation, but implemented elsewhere as well) represent a *mix-in* capability for transports that perform any internal buffering.

```thrift
(* interface *) service TBufferedTransport {
  i32 maxSize(),
  binary getBuffer(),
  i32 bufferPosition(),
  i32 bufferRemaining(),
  void consume(1: i32 bytes)
}
```

The following set of methods represent a mixin capability for transports that support the ability to move the read/write position in the underlying byte stream.

```thrift
(* interface *) service TSeekableTransport {
  i32 length(),
  i32 position(),
  void setPosition(1: i32 position)
}
```

The behavior of a seekable transport when the `position` is set beyond the value of `length()` is implementation specific.

## Server Transport API

The following is the server-side API that implements a listener loop for, and dispatches, client requests.

```thrift
(* interface *) service TServerTransport {
  void open(),
  void listen(),
  TTransport accept(),
  void close()
}
```
