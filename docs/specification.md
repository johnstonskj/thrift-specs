# Specification Details

The specifications in this repo are not intended to be formal, but a level of consistency helps the reader and so we use some conventions when describing the elements of the protocol, transport, and client/server layers.

## IDL used

The IDL used in the examples is based upon the Thrift IDL itself, however we treat the existing `service` as an interface intended to describe abstractions.

## BNF notation used in this document

All specifications will use ISO 14977 standard [Extended_Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form), summarized in the following table:


Usage	           | Notation    | Comment
-----------------|-------------|--------
definition	      | `=`         |
concatenation    |	`,`         |
termination	     | `;`         |
alternation      |	`|`         |
optional	        | `[ ... ]`   | zero or one
repetition	      | `{ ... }`   | zero to many
grouping	        | `( ... )`   |
terminal string	 | `" ... "`   |
terminal string	 | `' ... '`   | Unused in this site.
comment	         | `(* ... *)` |
special sequence	| `? ... ?`   | Used for extension specification.
exception	       | `-`         |


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
