# proglog

My attempt at creating a distributed service with Go.

This is a personal learning project that follows along with
[**Distributed Services with Go**](https://pragprog.com/titles/tjgo/distributed-services-with-go/)
by Travis Jeffery (The Pragmatic Bookshelf). I'm working through the book to get
hands-on experience with commit logs, gRPC/protobuf, and distributed-systems
patterns in Go.

> **Status: work in progress.** Code lands chapter by chapter as I read along, so
> some pieces are incomplete and a few tests don't yet compile. Expect rough
> edges.

## What's implemented so far

The project currently covers the book's early chapters:

- **JSON/HTTP commit log server** (`internal/server`, `cmd/server`) — a simple
  HTTP API over an in-memory log, with `produce` (append) and `consume` (read)
  endpoints backed by an append-only, offset-addressed record list.
- **Protobuf API definition** (`api/v1`) — a `Record` message defined in
  `log.proto`, with generated Go code (`log.pb.go`).
- **Persisted store** (`internal/log/store.go`) — an append-only, length-prefixed
  file store with buffered writes, the foundation for the on-disk log. Its tests
  (`store_test.go`) are still being written and are the main WIP piece.

### Layout

```
api/v1/          protobuf definitions + generated code (Record message)
cmd/server/      main entry point; starts the HTTP server on :8080
internal/server/ JSON/HTTP server, in-memory commit log
internal/log/    on-disk store (append-only, length-prefixed)
```

## Tech stack

- **Go** (module targets Go 1.17)
- **Protocol Buffers / protobuf-go** for the API schema (`google.golang.org/protobuf`)
- **gorilla/mux** for HTTP routing
- **testify** for tests

## Building and running

Compile the protobuf definitions (requires `protoc` and the Go plugins on your
`PATH`):

```sh
make compile
```

Build everything:

```sh
go build ./...
```

Run the HTTP server (listens on `:8080`):

```sh
go run ./cmd/server
```

### Trying the HTTP API

Produce a record (the value is a base64-encoded byte string):

```sh
curl -X POST localhost:8080 \
  -d '{"record": {"value": "TGV0J3MgR28gIzEK"}}'
```

Consume a record by offset:

```sh
curl -X GET localhost:8080 \
  -d '{"offset": 0}'
```

## Tests

```sh
make test     # go test -race ./...
```

Note: as mentioned above, some tests are still being written and may not compile
yet while the corresponding code is in progress.

## Credit

All design and structure come from
[*Distributed Services with Go*](https://pragprog.com/titles/tjgo/distributed-services-with-go/)
by Travis Jeffery. This repo is for my own learning; please support the author by
buying the book.
