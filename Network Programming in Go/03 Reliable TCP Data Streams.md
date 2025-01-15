---
tags:
  - go
  - go-networking
created: 2024-11-09
---
### Listeners.
A listener in Go, is specifically a server-side construct. It listens for incoming connections on a specific address and port, which clients can connect to.

Typically, you have a for loop that continuously listens for incoming connections and when it accepts, it will spin that connection into its own goroutine.

```go
for {
	conn, err := listener.Accept()
	if err != nil {
		return err
	}
	go func(c net.Conn) {
		defer c.Close()
		// Do thing here
	}(conn)
}
```

### Dial.
`Dial` is used on the client-side to connect to a server by knowing it's IP address and port.

```go
conn, err := net.Dial("tcp", listener.Addr().String())
```

### `net` Package Errors.
The `net` package implements the `net.Error` interface which gives you more specific error responses like `Timeout` and `Temporary`. You will need to use type assertions to verify that the error type is of type `net.Error` though.

