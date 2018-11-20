# Transport API and Behavior

```
interface TTransport {
  open()
  close()
  is-open() : bool
  read() : value
  write(v : value)
  flush()
}
```

```
interface TServerTransport {
  open()
  listen()
  accept()
  close()
}
```
