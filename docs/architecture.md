# Thrift Architecture

The overall architecture for Thrift is a little more complex than the 4-layer *networking stack* usually described. For one thing we need to include the client and IDL, and for another the original stack doesn't differentiate between standard transports/protocols and wrappers that add additional functionality.

```
  ,----------------------,
  | Server               |
  | (single-threaded,    |
  |   event-driven etc)  |
  |----------------------| ,----------------------,    ,----------------------,
  | Processor            | | Client               | <= | IDL                  |
  | (compiler generated) | | (compiler generated) | <= | (language generator) |
  |---------------------,'-'----------------------|    '----------------------'
  |                     | Wrappers                | 
  | Protocol            | (Multiplexed etc)       |
  |                     '-------------------------|
  | (JSON, compact etc)                           |
  |---------------------,-------------------------|
  |                     | Wrappers                |
  | Transport           | (Framed, ZLib etc)      |
  |                     '-------------------------|
  | (raw TCP, HTTP etc)                           |
  '-----------------------------------------------'
```


