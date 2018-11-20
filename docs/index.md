# Apache Thrift Specs

## Architecture

From [Concepts](https://thrift.apache.org/docs/concepts)

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
* Commonly Supported Transports

## Protocols

The Protocol abstraction defines a mechanism to map in-memory data structures to a wire-format. In other words, a protocol specifies how datatypes use the underlying Transport to encode/decode themselves. Thus the protocol implementation governs the encoding scheme and is responsible for (de)serialization. Some examples of protocols in this sense include JSON, XML, plain text, compact binary etc.

* [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-common.md)
* *End-Point* and *Wrapper* Protocols
* Commonly Supported Protocols

## Processors

TBD

## Servers

TBD

## Interface Definition Language

TBD

## References

1. [Original Thrift Whitepaper](http://thrift.apache.org/static/files/thrift-20070401.pdf)
2. [Apache Docs](https://thrift.apache.org/docs/)
2. [Apache-Hosted Wiki](https://wiki.apache.org/thrift)
3. [Apache-Hosted Tutorial](http://wiki.apache.org/thrift/Tutorial)
4. [The Missing Guide](https://diwakergupta.github.io/thrift-missing-guide)
