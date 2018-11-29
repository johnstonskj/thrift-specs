# Apache Thrift Specifications

## Architecture

The following, commonly referenced diagram, is taken from the Apache Thrift [Concepts](https://thrift.apache.org/docs/concepts) document.

```
  +-------------------------------------------+
  | Server                                    |
  | (single-threaded, event-driven etc)       |
  +-------------------------------------------+
  | Processor                                 |
  | (compiler generated)                      |
  +-------------------------------------------+
  | Protocol                                  |
  | (JSON, compact etc)                       |
  +-------------------------------------------+
  | Transport                                 |
  | (raw TCP, HTTP etc)                       |
  +-------------------------------------------+
```

However, there is more to the overall picture and we describe a more complete picture in the [Architecture](https://johnstonskj.github.io/thrift-specs/architecture) page.

The following sections will outline details on the specific layers, with specifications following these [specification guidelines](https://johnstonskj.github.io/thrift-specs/specification).

## Transports

The Transport layer provides a simple abstraction for reading/writing from/to the network. This enables Thrift to decouple the underlying transport from the rest of the system (serialization/deserialization, for instance).

* [Transport API and Behavior](https://johnstonskj.github.io/thrift-specs/transport-api)

### Transport Types Supported

Type | Required | Comments
-----|----------|---------
Sockets  | Minimal required | TCP and Unix domain
[Buffered](https://johnstonskj.github.io/thrift-specs/transport-buffered) | Minimal required | *wrapper*
[Framed](https://johnstonskj.github.io/thrift-specs/transport-buffered)   | Minimal required | *wrapper*
HTTP Client | Minimal recommended |
HTTP Server | Other recommended |
Pipes | Other recommended |
Named Pipes | Other recommended | Where it makes sense

> We use the term *wrapper* in these documents to describe a capability that wraps another of the same type and enhances it's behavior in some manner.

## Protocols

The Protocol abstraction defines a mechanism to map in-memory data structures to a wire-format. In other words, a protocol specifies how datatypes use the underlying Transport to encode/decode themselves. Thus the protocol implementation governs the encoding scheme and is responsible for (de)serialization. Some examples of protocols in this sense include JSON, XML, plain text, compact binary etc.

* [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-api)

### Protocol Types Supported

Type | Required | Comments
-----|----------|---------
[Binary](https://johnstonskj.github.io/thrift-specs/protocol-binary)    | Minimal required |
[Multiplex](https://johnstonskj.github.io/thrift-specs/protocol-multiplex) | Minimal required | *wrapper*
[JSON](https://johnstonskj.github.io/thrift-specs/protocol-json) | Minimal recommended |
[Compact](https://johnstonskj.github.io/thrift-specs/protocol-compact) | Other recommended | required for [Parquet](https://parquet.apache.org/)

## Processors

A Processor encapsulates the ability to read data from input streams and write to output streams. The input and output streams are represented by Protocol objects. The Processor interface is extremely simple.

### Messaging Considerations

## Servers

A Server pulls together all of the various features described above:

* Create a transport
* Create input/output protocols for the transport
* Create a processor based on the input/output protocols
* Wait for incoming connections and hand them off to the processor


### Server Types Supported

Type | Required | Comments
-----|----------|---------
Simple    | Minimal required |
Non-Blocking    | Other recommended |
Threaded, Thread Pool | Other recommended | and/or

## Interface Definition Language

TBD

## References

1. [Original Thrift Whitepaper](http://thrift.apache.org/static/files/thrift-20070401.pdf)
2. [Apache Docs](https://thrift.apache.org/docs/)
2. [Apache-Hosted Wiki](https://wiki.apache.org/thrift)
3. [Apache-Hosted Tutorial](http://wiki.apache.org/thrift/Tutorial)
4. [The Missing Guide](https://diwakergupta.github.io/thrift-missing-guide)
5. [Official Thrift Source](https://github.com/apache/thrift)
5. [ThriftPy2 - Python Implementation](https://github.com/Thriftpy/thriftpy2)
