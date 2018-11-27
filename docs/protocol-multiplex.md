# Multiplex Protocol

This specification describes the multiplex **protocol wrapper** for Thrift, an required component of the minimal language support for Thrift. The purpose of this wrapper is to allow a client to send messages to multiple services over the same transport in a seamless manner. To do this the wrapper extends the usual method name into a `service:method` format to identify the target service and method explicitly. On the server side a corresponding *multiplex processor* is specified that can decode and dispatch messages to the correct service implementation.

This specification refers to the [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-api) which defines protocol-agnostic and default behavior.

## BNF For Multiplex Protocol

Whereas the BNF in [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-api) describes `method-name` in the following manner:

```ebnf
method-name    = string ;
```

This wrapper ensures that the encoded form is extended in the following manner:

```ebnf
method-name    = service-name , ":" , service-method ;
service-name   = string ;
service-method = string ;
```

## Basic Type Encoding

Unaffected.

## Message Encoding

Unaffected.

## Struct and Union Encoding

Unaffected.

## List and Set Encoding

Unaffected.

## Map Encoding

Unaffected.

## Examples

TBD

## References

* [THRIFT-564: Support for Multiplexing Services on any Transport, Protocol and Server](https://issues.apache.org/jira/browse/THRIFT-563)
