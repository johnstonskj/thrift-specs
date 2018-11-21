# Transport API and Behavior

```
interface TTransport {
  void open(),
  void close(),
  value read(),
  write(1 : value v),
  void flush()
}
```

```
interface TServerTransport {
  void open(),
  void listen(),
  void accept(),
  void close()
}
```
