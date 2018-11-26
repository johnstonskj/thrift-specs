# Specification Details

The specifications in this repo are not intended to be formal, but a level of consistency helps the reader and so we use some conventions when describing the elements of the protocol, transport, and client/server layers.

## IDL used

The IDL used in the examples is based upon the Thrift IDL itself, however we add an `interface` keyword with an identical lexical form to the existing `service` but intended to describe abstractions.

## BNF notation used in this document

The following BNF notation is used:

* a plus `+` appended to an item represents repetition; the item is repeated 1 or more times
* a star `*` appended to an item represents optional repetition; the item is repeated 0 or more times
* a pipe `|` between items represents choice, the first matching item is selected
* parenthesis `(` and `)` are used for grouping multiple items

## Protocol Specifications

Each protocol will be specified according to a common format, if it is not considered a *wrapper* protocol it will have the following structure (see [Protocol Template](https://johnstonskj.github.io/thrift-specs/protocol-template)):

* BNF summarizing the structure
* A description of basic type encodings
* A description of the message encoding
* A description of the struct and union encoding
  * A description of field encodings
  * A description of stop field handling
* A description of the list and set encodings
* A description of the map encoding
* Examples
* References

Wrapper protocols will rarely have a full BNF description but will describe how they affect any protocol that they wrap.
