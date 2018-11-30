# HTTP Transports

This specification describes the HTTP transports, the HTTP client is a recommended component of the minimal language support for Thrift whereas the HTTP Server is simply recommended. 

This specification refers to the [Transport API and Behavior](https://johnstonskj.github.io/thrift-specs/transport-api) which defines transport-agnostic and default behavior.

## HTTP

The HTTP transport is relatively simple, an entire encoded message is sent from client to server in the entity body of a `POST` method call. This entity is given the content type `application/x-thrift` and any further encoding/decoding is performed based on standard transport/protocol behavior. The intention is to keep the minimal requirements minimal, with a very simple mapping to standard HTTP concepts. 

### Headers

Header Name | Value | Comments
------------|-------|---------
`Content-Type` | `application/x-thrift` |
`Accept` | `application/x-thrift` |

Additionally:

* `Connection` - C++ allows `Keep-Alive`
* `Transfer-Encoding` - C++
* `X-Forwarded-For` - C++

C++ also supports CORS pre-flight option check as well as `Access-Control-*` headers on POST response.

### Client Considerations

### Server Considerations

