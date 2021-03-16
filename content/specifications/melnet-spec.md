---
title: "Melnet specification (WIP)"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 10
draft: false
# search related keywords
keywords: [""]
---

**Melnet** is an almost trivial request/response protocol that underpins Themelio's peer-to-peer network.

## Basic semantics

Each peer-to-peer network is identified with a **netname**, which is sent with all requests.

The only kind of communication pattern is a simple request-response, which is sent over a TCP connection. The way the connection is used is similar to HTTP/1.1: the client sends a request over a connection, waiting for the response before sending another one. Parallel requests are done by opening multiple connections, and connection pooling should be used to save resources.

In a single request-response roundtrip:

- The **request** contains a **verb**, a string representing a RPC function call (say, `send_tx`), as well as arbitrary bytes representing the **body**. These bytes conventionally represent a value bincode-encoded the same way as Themelio state elements are. This is represented by the following bincode-encoded structure:
  ```rust
  pub struct RawRequest {
      pub proto_ver: u8,
      pub netname: String,
      pub verb: String,
      pub payload: Vec<u8>,
  }
  ```
  which is sent over the wire preceded by its encoded length, as a 32-bit (4-byte) big-endian number.
- The **response** is a bincode-encoded structure:
  ```rust
  pub struct RawResponse {
      pub kind: String,
      pub body: Vec<u8>,
  }
  ```
  which is also sent as a length-prefixed value. Here, `kind` is one of:
  - `"Ok"`, indicating success. `body` will contain the response to the RPC call.
  - `"NoVerb"`, indicating that no such verb exists.
  - `"Err"`, indicating an error. `body` will contain a descriptive UTF-8 string.

## Establishing a broadcast mesh
