# JSON Protocol


This specification describes the JSON wire encoding for Thrift, a recommended component of the minimal language support for Thrift.

This specification is based upon the [C++ implementation header](https://github.com/apache/thrift/blob/master/lib/cpp/src/thrift/protocol/TJSONProtocol.h) in GitHub.

This specification refers to the [Protocol API and Behavior](https://johnstonskj.github.io/thrift-specs/protocol-api) which defines protocol-agnostic and default behavior.

## BNF For JSON Protocol

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
field-id         = i16 (* as string *) ;
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

Thrift messages are represented as JSON arrays, with the protocol version number (currently `1`), the message name, the message type, and the sequence ID as the first 4 elements.

## Struct and Union Encoding

Thrift structs are represented as JSON objects. Fields are encoded as object properties, the key is the `field-id`, string encoded, and the field is encoded as described in the next section. Note that JSON keys can only be strings and therefore `field-id`s are serialized as strings.

```json
{"1":{...},"2":{...},...}
```

### Field Encoding

Each field is encoded as a JSON object, that object has a single key and a single value. The key is a short string identifier  `type-id` (see table below).

Type   | `type-id`   | Comments
-------|----------|---------
bool   | "tf"    |
byte   | "i8"    |
double | "dbl"   | See below.
int16  | "i16"   |
int32  | "i32"   |
int64  | "i64"   |
string | "str"   | With appropriate escaping.
binary | "str"   | Encoded into Base64.
struct | "rec"   | (for "record").
map    | "map"   |
list   | "lst"   |
set    | "set"   |

The following example shows the result of a struct encoding.

```json
{
  "1": {
    "str": "some message"
  },
  "2": {
    "i32": 200
  }
}
```

### Stop Field Handling

No stop field is required as each struct is terminated by an enclosing brace, "}".

## List and Set Encoding

Thrift lists and sets are represented as JSON arrays, with the first element of the JSON array being the string identifier for the Thrift element type and the second element of the JSON array being the count of the Thrift elements. The Thrift elements then follow.

```json
["str",2,"hello","world"]
```

## Map Encoding

Thrift maps are represented as JSON arrays, with the first two elements of the JSON array being the string identifiers for the Thrift key type and value type, followed by the count of the Thrift pairs, followed by a JSON object containing the key-value pairs. Note that JSON keys can only be strings, which means that the key type of the Thrift map should be restricted to numeric or string types -- in the case of numerics, they are serialized as strings.

```json
["str","str",2,{"msg":"hello"},{"to":"world"}]
```

## Examples

An example call message with no struct content.

```json
[1,"method",1,99]
```

An example response encoding a UUID as a binary entity.

```json
[2,"method",2,99,{"1":{"i8":2},"2":{"str":"NjFFMEE0RkItQzNBMy00ODBGLTk3MjgtODc4MDg3M0Q1OTVFCg=="}}]
```

An example exception response.

```json
[1,"method",3,{"1":{"str":"wrong method name: 'method'"},"2":{"i32":3}}]
```

## References

TBD
