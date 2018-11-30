# HTTP Transports

This specification describes the HTTP transports, the HTTP client is a recommended component of the minimal language support for Thrift whereas the HTTP Server is simply recommended. 

This specification refers to the [Transport API and Behavior](https://johnstonskj.github.io/thrift-specs/transport-api) which defines transport-agnostic and default behavior.

## HTTP Client

### Headers

Header Name | Value | Comments
------------|-------|---------
`Content-Type` | `application/x-thrift` |
`Accept` | `application/x-thrift` |

## HTTP Server
