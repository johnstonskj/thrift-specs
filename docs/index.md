# Apache Thrift Specs

## Architecture

TRhe following, commonly referenced diagram is taken from the Apache Thrift [Concepts](https://thrift.apache.org/docs/concepts) document.

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

However, there is more to the overall picture, see [Architecture](https://johnstonskj.github.io/thrift-specs/architecture.md).

## Transports

The Transport layer provides a simple abstraction for reading/writing from/to the network. This enables Thrift to decouple the underlying transport from the rest of the system (serialization/deserialization, for instance).

* [Transport API and Behavior](https://johnstonskj.github.io/thrift-specs/transport-common.md)
* *End-Point* and *Wrapper* Transports

Type | Required | Comments
-----|----------|---------
Sockets  | Minimal required | TCP and Unix domain
Buffered | Minimal required |
Framed   | Minimal required |
HTTP Client | Minimal recommended |
HTTP Server | Other recommended |
Pipes | Other recommended |
Named Pipes | Other recommended | Where it makes sense

## Protocols

The Protocol abstraction defines a mechanism to map in-memory data structures to a wire-format. In other words, a protocol specifies how datatypes use the underlying Transport to encode/decode themselves. Thus the protocol implementation governs the encoding scheme and is responsible for (de)serialization. Some examples of protocols in this sense include JSON, XML, plain text, compact binary etc.

* [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-common.md)

Type | Required | Comments
-----|----------|---------
Binary    | Minimal required |
Multiplex | Minimal required |
JSON | Minimal recommended |
Compact | Other recommended | (required for [Parquet](https://parquet.apache.org/))

## Processors

TBD

## Servers

TBD

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
