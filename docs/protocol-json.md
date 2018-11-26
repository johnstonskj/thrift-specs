# Protocol {protocol-name}

## BNF For {protocol-name}

; From <https://github.com/apache/thrift/blob/master/lib/cpp/src/thrift/protocol/TJSONProtocol.h>
;
; * Implements a protocol which uses JSON as the wire-format.
; *
; * Thrift types are represented as described below:
; *
; * 1. Every Thrift integer type is represented as a JSON number.
; *
; * 2. Thrift doubles are represented as JSON numbers. Some special values are
; *    represented as strings:
; *    a. "NaN" for not-a-number values
; *    b. "Infinity" for positive infinity
; *    c. "-Infinity" for negative infinity
; *
; * 3. Thrift string values are emitted as JSON strings, with appropriate
; *    escaping.
; *
; * 4. Thrift binary values are encoded into Base64 and emitted as JSON strings.
; *    The readBinary() method is written such that it will properly skip if
; *    called on a Thrift string (although it will decode garbage data).
; *
; *    NOTE: Base64 padding is optional for Thrift binary value encoding. So
; *    the readBinary() method needs to decode both input strings with padding
; *    and those without one.
; *
; * 5. Thrift structs are represented as JSON objects, with the field ID as the
; *    key, and the field value represented as a JSON object with a single
; *    key-value pair. The key is a short string identifier for that type,
; *    followed by the value. The valid type identifiers are: "tf" for bool,
; *    "i8" for byte, "i16" for 16-bit integer, "i32" for 32-bit integer, "i64"
; *    for 64-bit integer, "dbl" for double-precision loating point, "str" for
; *    string (including binary), "rec" for struct ("records"), "map" for map,
; *    "lst" for list, "set" for set.
; *
; * 6. Thrift lists and sets are represented as JSON arrays, with the first
; *    element of the JSON array being the string identifier for the Thrift
; *    element type and the second element of the JSON array being the count of
; *    the Thrift elements. The Thrift elements then follow.
; *
; * 7. Thrift maps are represented as JSON arrays, with the first two elements
; *    of the JSON array being the string identifiers for the Thrift key type
; *    and value type, followed by the count of the Thrift pairs, followed by a
; *    JSON object containing the key-value pairs. Note that JSON keys can only
; *    be strings, which means that the key type of the Thrift map should be
; *    restricted to numeric or string types -- in the case of numerics, they
; *    are serialized as strings.
; *
; * 8. Thrift messages are represented as JSON arrays, with the protocol
; *    version #, the message name, the message type, and the sequence ID as
; *    the first 4 elements.
; *
; * More discussion of the double handling is probably warranted. The aim of
; * the current implementation is to match as closely as possible the behavior
; * of Java's Double.toString(), which has no precision loss.  Implementors in
; * other languages should strive to achieve that where possible. I have not
; * yet verified whether std::istringstream::operator>>, which is doing that
; * work for me in C++, loses any precision, but I am leaving this as a future
; * improvement. I may try to provide a C component for this, so that other
; * languages could bind to the same underlying implementation for maximum
; * consistency.

```ebnf
<tbd>
```

## Basic Type Encoding

Type   | Format | Comments
-------|--------|---------
bool   | TBD |
byte   | TBD |
double | TBD |
int16  | TBD |
int32  | TBD |
int64  | TBD |
string | TBD |
list   | TBD |
set    | TBD |
map    | TBD |
struct | TBD |

## Message Encoding

TBD

## Struct and Union Encoding

TBD

### Field Encoding

TBD

### Stop Field Handling

TBD

## List and Set Encoding

TBD

## Map Encoding

TBD

## Examples

TBD

## References

TBD

