# Buffered and Framed Transports

This specification describes the buffered and framed wrapper transports, both are required components of the minimal language support for Thrift. Any buffering performed by these wrappers is independent of any *intrinsic* buffering provided by any implementation of `TBufferedTransport`.

This specification refers to the [Transport API and Behavior](https://johnstonskj.github.io/thrift-specs/transport-api) which defines transport-agnostic and default behavior.

## Buffered Transport

This wrapper provides a capability to buffer read and write operations over another *wrapped* transport. 

The behavior of the `flush()` procedure is undefined when writing to a buffered transport; in general it is assumed that writes *only* occur when the buffer is full, or on `close()` if the buffer is not empty.

In terms of configuration an implementation may choose to support any or all of the following buffer size controls:

* A single fixed, internal, buffer size.
* A single buffer size set when the transport is initialized.
* Separate read and write buffer sizes set when the transport is initialized.

From the C++ header:

```c++
TBufferedTransport(stdcxx::shared_ptr<TTransport> transport);
TBufferedTransport(stdcxx::shared_ptr<TTransport> transport, uint32_t sz);
TBufferedTransport(stdcxx::shared_ptr<TTransport> transport, uint32_t rsz, uint32_t wsz);
```

## Framed Transport

This wrapper provides a capability to ensure the read of an entire message (or defined components thereof) by preceding the message with a 4-byte integer representing the number of bytes to be read - the frame. 

The procedure `flush()` (and `close()` is expected to call `flush()`) is expected to be the point at which a frame is written to the wrapped transport from the internal buffer.

> Note that a default or base implementation of `TProtocol` may insert a call to `flush()` from within `writeMessageEnd()`.
