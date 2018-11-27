# Protocol {protocol-name}

From [C++ Header](https://github.com/apache/thrift/blob/master/lib/cpp/src/thrift/protocol/TJSONProtocol.h)

This specification refers to the [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-api) which defines protocol-agnostic and default behavior.

## BNF For {protocol-name}

```ebnf
message          = "[" , protocol-version , message-name , message-type , sequence-id
                       , struct "]" 
                 ;
protocol-version = number ;
message-name     = string ;
sequence-id      = number ;
struct           = "{" , [ field-list ] , "}" ;
field-list       = field , ","
                 | field ;
field            = field-id , ":" , field-value ;
field-id         = i16 ;
field-value      = "{" , type-id , ":" , value , "}" ;
type-id          = "tf" | "i8" | "i16" | "i32" | "i64" 
                 | "dbl" | "str" | "rec" | "map" | "lst" | "set" ;
value            = boolean | number | string 
                 | struct | list | set | map ;
list             = "[" , type-id , "," , count , list-values , "]" ;
count            = number ;
list-values      = { "," , value } ;
set              = list ;
map              = "[" , type-id (* key *)   , "," 
                       , type-id (* value *) , "," 
                       , count , "," 
                       , { map-values } , "]" ;
map-values       = "{" , field-key , ":" , value , "}" ;
field-key        = string ;
```

## Basic Type Encoding

Type   | Format   | Comments
-------|----------|---------
bool   | boolean  |
byte   | number   |
double | number   | See below.
int16  | number   |
int32  | number   |
int64  | number   |
string | string   | With appropriate escaping.
binary | string   | Encoded into Base64.

Notes:

* Thrift doubles are represented as JSON numbers. Some special values are represented as strings:
  * "NaN" for not-a-number values
  * "Infinity" for positive infinity
  * "-Infinity" for negative infinity
* Base64 padding is optional for Thrift binary value encoding. So the `readBinary()` method needs to decode both input strings with padding and those without one.

More discussion of the double handling is probably warranted. The aim of the C++ implementation is to match as closely as possible the behavior of Java's `Double.toString()`, which has no precision loss.  Implementors in other languages should strive to achieve that where possible. 

## Message Encoding

Thrift messages are represented as JSON arrays, with the protocol version #, the message name, the message type, and the sequence ID as the first 4 elements.

## Struct and Union Encoding

Thrift structs are represented as JSON objects, with the field ID as the key, and the field value represented as a JSON object with a single key-value pair. The key is a short string identifier for that type, followed by the value. The valid type identifiers are: "tf" for bool, "i8" for byte, "i16" for 16-bit integer, "i32" for 32-bit integer, "i64" for 64-bit integer, "dbl" for double-precision loating point, "str" for string (including binary), "rec" for struct ("records"), "map" for map, "lst" for list, "set" for set.

### Field Encoding

TBD

### Stop Field Handling

TBD

## List and Set Encoding

Thrift lists and sets are represented as JSON arrays, with the first element of the JSON array being the string identifier for the Thrift element type and the second element of the JSON array being the count of the Thrift elements. The Thrift elements then follow.

## Map Encoding

Thrift maps are represented as JSON arrays, with the first two elements of the JSON array being the string identifiers for the Thrift key type and value type, followed by the count of the Thrift pairs, followed by a JSON object containing the key-value pairs. Note that JSON keys can only be strings, which means that the key type of the Thrift map should be restricted to numeric or string types -- in the case of numerics, they are serialized as strings.

## Examples

An example call message with no struct content.

```json
[1,"method",1,99]
```

An example exception response.

```json
[1,"method",3,{"1":{"str":"wrong method name: 'method'"},"2":{"i32":3}}]
```

## References

TBD
