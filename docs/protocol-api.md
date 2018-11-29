# Protocol API and Behavior

This specification describes the generic protocol layer interface and behavior, not any one specific implementation. The protocol layer provides an interface that supports the encoding and decoding of values over a *transport*. Different languages may provide idiomatic versions of the API described here, although the general order of operations and behavior should be preserved.

## Encoding-Agnostic Structure

The following EBNF description of the protocol, edited from [the Thrift GitHub docs](https://github.com/apache/thrift/edit/master/doc/specs/thrift-protocol-spec.md), describes the protocol in general terms without any encoding or other representational details. Other specifications should use this EBNF as a model when describing their specific encoding rules.

```ebnf
message        = message-begin , struct , message-end
               ;
message-begin  = method-name , message-type , message-seqid
               ;
method-name    = string
               ;
message-type   = T_CALL | T_REPLY | T_EXCEPTION | T_ONEWAY
               ;
message-seqid  = i32
               ;
struct         = struct-begin , { field } , field-stop , struct-end
               ;
struct-begin   = struct-name
               ;
struct-name    = string
               ;
field-stop     = T_STOP
               ;
field          = field-begin , field-data , field-end
               ;
field-begin    = field-name , field-type , field-id
               ;
field-name     = string
               ;
field-type     = T_BOOL | T_BYTE | T_I8 | T_I16 | T_I32 | T_I64 | T_DOUBLE
               | T_STRING | T_BINARY | T_STRUCT | T_MAP | T_SET | T_LIST
               ;
field-id       = i16
               ;
field-data     = i8 | i16 | i32 | i64 | double | string | binary
               | struct | map | list | set
               ;
map            = map-begin , { field-datum } , map-end
               ;
map-begin      = map-key-type , map-value-type , map-size
               ;
map-key-type   = field-type
               ;
map-value-type = field-type
               ;
map-size       = i32
               ;
list           = list-begin , { field-data } , list-end
               ;
list-begin     = list-elem-type , list-size
               ;
list-elem-type = field-type
               ;
list-size      = i32
               ;
set            = set-begin , { field-data } , set-end
               ;
set-begin      = set-elem-type , set-size
               ;
set-elem-type  = field-type
               ;
set-size       = i32 ;
```

They key point to notice is that ALL messages are just one wrapped <struct>. Depending upon the message type, the <struct> can be interpreted as the argument list to a function, the return value of a function, or an exception. This means that an encoding that has no explicit message termination can assume that the termination of the struct immediately after the message header terminates the message itself.

## Messages

A message header, `message-begin`, contains:

* `method-name`, a `T_STRING` (can be empty).
* `message-type`, a message types, one of `T_CALL`, `T_REPLY`, `T_EXCEPTION` and `T_ONEWAY`.
* `message-seqid`, a `T_I32`.

The *sequence id* is a simple message id assigned by the client. The server will use the same sequence id in the
message of the response. The client uses this number to detect out of order responses. Each client has an int32 field
which is increased for each message. The sequence id simply wraps around when it overflows.

The *method name* indicates the service method name to invoke. The server copies the name in the response message.

> When the *[multiplexed protocol](https://johnstonskj.github.io/thrift-specs/protocol-multiplex)* is used, the name contains the service name, a colon `:` and the method name. The multiplexed protocol is not compatible with other protocols.

The *message type* indicates what kind of message is sent. Clients send requests with TMessages of type `T_CALL` or
`T_ONEWAY` (step 1 in the protocol exchange). Servers send responses with messages of type `T_EXCEPTION` or `T_REPLY` (step
3). See [RPC Message Exchange](#RPC-Message-Exchange) for more details.

Type `T_REPLY` is used when the service method completes normally. That is, it returns a value or it throws one of the
exceptions defined in the Thrift IDL file.

Type `T_EXCEPTION` is used for other exceptions. That is: when the service method throws an exception that is not declared
in the Thrift IDL file, or some other part of the Thrift stack throws an exception. For example when the server could
not encode or decode a message or struct.

In the Java implementation (0.9.3) there is different behavior for the synchronous and asynchronous server. In the async
server all exceptions are send as a `TApplicationException` (see 'Response struct' below). In the synchronous Java
implementation only (undeclared) exceptions that extend `TException` are send as a `TApplicationException`. Unchecked
exceptions lead to an immediate close of the connection.

Type `T_ONEWAY` is only used starting from Apache Thrift 0.9.3. Earlier versions do _not_ send messages of type `T_ONEWAY`,
even for service methods defined with the `oneway` modifier.

When client sends a request with type `T_ONEWAY`, the server must _not_ send a response (steps 3 and 4 are skipped). Note
that the Thrift IDL enforces a return type of `T_VOID` and does not allow exceptions for oneway services.

## Basic Types

T_*ID*     | ID | IDL Type | Comments
-----------|----|----------|-----------------------------------
`T_STOP`   | 0  | *none*   | Used in the protocol for struct decoding.
`T_VOID`   | 1  | *none*   | Used in IDL.
`T_BOOL`   | 2  | `bool`   | Boolean value, `true` or `false`.
`T_BYTE`   | 3  | `byte`   | A single signed 8-bit byte.
`T_I8`     | 3  | `i8`     | A synonym for `T_BYTE`.
`T_I16`    | 6  | `i16`    | 16-bit signed integer.
`T_I32`    | 8  | `i32`    | 32-bit signed integer.
`T_I64`    | 10 | `i64`    | 64-bit signed integer.
`T_DOUBLE` | 4  | `double` | IEEE 64-bit floating point.
`T_STRING` | 11 | `string` | Character string.
`T_BINARY` | 11 | `binary` | String of `T_BYTE`.

Notes:

* Character string may be UTF-7 or UTF-8.
* Unless otherwise specified in a protocol, enumeration values are encoded as `T_I32` values.

## Structured Types

T_*ID*     | ID | Comments
-----------|----|-----------------------------------
`T_STRUCT` | 12 |
`T_MAP`    | 13 |
`T_SET`    | 14 |
`T_LIST`   | 15 |

Unless otherwise specified in a protocol, a **union** is encoded in the same way as a structure, except that it enforces the rule that one, and only one, field may be present at any time, as in:
  
```ebnf
union = struct-begin field field-stop struct-end ;
```

Unless otherwise specified in a protocol, an **exception** is encoded in the same way as a structure.

<!--
## Additional Types

ID | Implementation | T_*ID*
---|----------------|-------
9  | C++            | `T_U64`
16 | PyThrift2, C++ | `T_UTF8`
16 | Java           | `ENUM`
17 | PyThrift2, C++ | `T_UTF16`

-->

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

**Example**

```racket
(define (write-message-over transport)
  (define p (make-protocol-encoder transport))
  ((encoder-message-begin p) (message-header "mthod" 7 9))

  ((encoder-struct-begin p) "unused")

  ((encoder-field-begin p) (field-header "name" type-string 1))
  ((encoder-string p) "simon")
  ((encoder-field-end p))

  ((encoder-field-begin p) (field-header "age" type-byte 2))
  ((encoder-byte p) 48)
  ((encoder-field-end p))

  ((encoder-field-begin p) (field-header "brilliant?" type-bool 3))
  ((encoder-boolean p) #f)
  ((encoder-field-end p))

  ((encoder-field-stop p))

  ((encoder-struct-end p))

  ((encoder-map-begin p) (map-header type-string type-int32 0))
  ((encoder-string p) "key")
  ((encoder-string p) "value")
  ((encoder-string p) "key2")
  ((encoder-int32 p) 101)
  ((encoder-int32 p) 202)
  ((encoder-string p) "value?")
  ((encoder-map-end p))

  ((encoder-message-end p)))
```

### Interface Types

```thrift
struct MessageHeader {
  1: string name,
  2: CallType type,
  3: i32 sequence
}

struct FieldHeader {
  1: string name,
  2: Type type,
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

### Exceptions

When the message is of type Exception the struct is encoded as if it was declared by the following IDL:

```thrift
exception TApplicationException {
  1: string message,
  2: i32 type
}
```

The following exception types are defined in the Java implementation (0.9.3):

Code | Type  | Usage in Java Implementation
-----|-------|--------
0    | unknown              | Used in case the type from the peer is unknown.
1    | unknown method       | Used in case the method requested by the client is unknown by the server.
2    | invalid message type | None found.
3    | wrong method name    | None found.
4    | bad sequence id      | Used internally by the client to indicate a wrong sequence id in the response.
5    | missing result       | Used internally by the client to indicate a response without any field (result nor exception).
6    | internal error       | Used when the server throws an exception that is not declared in the Thrift IDL file. 
7    | protocol error       | Used when something goes wrong during decoding. For example when a list is too long or a required field is missing. 
8    | invalid transform    | None found.
9    | invalid protocol     | None found.
10   | unsupported client type | None found.

[Protocol Template](https://johnstonskj.github.io/thrift-specs/protocol-template).

## RPC Message Exchange

Both the binary protocol and the compact protocol assume a transport layer that exposes a bi-directional byte stream,
for example a TCP socket. Both use the following exchange:

1. Client sends a `Message` (type `Call` or `Oneway`). The TMessage contains some metadata and the name of the method
   to invoke.
2. Client sends method arguments (a struct defined by the generate code).
3. Server sends a `Message` (type `Reply` or `Exception`) to start the response.
4. Server sends a struct containing the method result or exception.

The pattern is a simple half duplex protocol where the parties alternate in sending a `Message` followed by a struct.
What these are is described below.

Although the standard Apache Thrift Java clients do not support pipelining (sending multiple requests without waiting
for an response), the standard Apache Thrift Java servers do support it.
