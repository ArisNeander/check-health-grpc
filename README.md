# check-health-grpc(1)

![ci](https://github.com/grpc-ecosystem/grpc-health-probe/workflows/ci/badge.svg)
![GitHub all releases](https://img.shields.io/github/downloads/grpc-ecosystem/grpc-health-probe/total)


The `check-health-grpc` monitoring plugin allows you to query health of gRPC services that
expose service their status through the [gRPC Health Checking Protocol][hc].

This command-line utility makes a RPC to `/grpc.health.v1.Health/Check`. If it
responds with a `SERVING` status, the `grpc_health_probe` will exit with
success, otherwise it will exit with a non-zero exit code (documented below).

`check-health-grpc` is meant to be used for health checking gRPC applications in
[Kubernetes][k8s], using the [exec probes][execprobe].

**EXAMPLES**

```text
$ check-health-grpc -addr=localhost:5000
healthy: SERVING
```

```text
$ check-health-grpc -addr=localhost:9999 -connect-timeout 250ms -rpc-timeout 100ms
failed to connect service at "localhost:9999": context deadline exceeded
exit status 2
```

## Installation

**It is recommended** to use a version-stamped binary distribution:
  (Please note that binary distribution has not been set up for this fork yet)

- Choose a binary from the [Releases][rel] page.

Installing from source (not recommended):

- Make sure you have `git` and `go` installed.
- Run: `go get github.com/ArisNeander/check-health-grpc`
- This will compile the binary into your `$GOPATH/bin` (or `$HOME/go/bin`).

## Using the gRPC Health Checking Protocol

To make use of the `check-health-grpc`, your application must implement the
[gRPC Health Checking Protocol v1][hc]. This means you must to register the
`Health` service and implement the `rpc Check` that returns a `SERVING` status.

Since the Health Checking protocol is part of the gRPC core, it has
packages/libraries available for the languages supported by gRPC:

[[health.proto](https://github.com/grpc/grpc/blob/master/src/proto/grpc/health/v1/health.proto)]
[[Go](https://godoc.org/google.golang.org/grpc/health/grpc_health_v1)]
[[Java](https://github.com/grpc/grpc-java/blob/master/services/src/generated/main/grpc/io/grpc/health/v1/HealthGrpc.java)]
[[Python](https://github.com/grpc/grpc/tree/master/src/python/grpcio_health_checking)]
[[C#](https://github.com/grpc/grpc/tree/master/src/csharp/Grpc.HealthCheck)/[NuGet](https://www.nuget.org/packages/Grpc.HealthCheck/)]
[[Ruby](https://www.rubydoc.info/gems/grpc/Grpc/Health/Checker)] ...

Most of the languages listed above provide helper functions that hides
implementation details. This eliminates the need for you to implement the
`Check` rpc yourself.

## Example: gRPC health checking on Kubernetes

Kubernetes does not natively support gRPC health checking since it does not
favor one RPC framework over another. Similarly, HTTP health probes Kubernetes
has is not sufficient to craft a valid gRPC request. As a solution,
`grpc_health_probe` [can be used for Kubernetes][k8s] to health-check gRPC
servers running in the Pod.

You are recommended to use [Kubernetes `exec` probes][execprobe] and define
liveness and/or readiness checks for your gRPC server pods.

## Health Checking TLS Servers

If a gRPC server is serving traffic over TLS, or uses TLS client authentication
to authorize clients, you can still use `check-health-grpc` to check health
with command-line options:

| Option | Description |
|:------------|-------------|
| **`-tls`** | use TLS (default: false) |
| **`-tls-ca-cert`** | path to file containing CA certificates (to override system root CAs) |
| **`-tls-client-cert`** | client certificate for authenticating to the server |
| **`-tls-client-key`** | private key for for authenticating to the server |
| **`-tls-no-verify`** | use TLS, but do not verify the certificate presented by the server (INSECURE) (default: false) |
| **`-tls-server-name`** | override the hostname used to verify the server certificate |

## Other Available Flags

| Option | Description |
|:------------|-------------|
| **`-v`**    | verbose logs (default: false) |
| **`-connect-timeout`** | timeout for establishing connection |
| **`-rpc-timeout`** | timeout for health check rpc |
| **`-user-agent`** | user-agent header value of health check requests (default: check-health-grpc) |
| **`-service`** | service name to check (default: "") - empty string is convention for server health |
| **`-gzip`** | use GZIPCompressor for requests and GZIPDecompressor for response (default: false) |

**Example:**

1. Start the `route_guide` [example
   server](https://github.com/grpc/grpc-go/tree/be59908d40f00be3573a50284c3863f1a37b8528/examples/route_guide)
   with TLS by running:

       go run server/server.go -tls

2. Run `check-health-grpc` with the [CA
   certificate](https://github.com/grpc/grpc-go/blob/be59908d40f00be3573a50284c3863f1a37b8528/testdata/ca.pem)
   (in the `testdata/` directory) and hostname override the
   [cert](https://github.com/grpc/grpc-go/blob/be59908d40f00be3573a50284c3863f1a37b8528/testdata/server1.pem) is signed for:

      ```sh
      $ check-health-grpc -addr 127.0.0.1:10000 \
          -tls \
          -tls-ca-cert /path/to/testdata/ca.pem \
          -tls-server-name=x.test.youtube.com

      status: SERVING
      ```

## Exit codes

This utility returns Nagios return codes.

Used are:

| Exit Code | Description |
|:-----------:|-------------|
| **0** | success: rpc response is `SERVING`. (Status: OK)|
| **3** | failure: invalid command-line arguments  (Status: UNKNOWN)|
| **2** | failure: connection failed or timed out (Status: CRITICAL)|
| **2** | failure: rpc failed or timed out (Status: CRITICAL)|
| **2** | failure: rpc successful, but the response is not `SERVING` (Status: CRITICAL)|

----

This is not an official Google project.

[hc]: https://github.com/grpc/grpc/blob/master/doc/health-checking.md
[k8s]: https://kubernetes.io/blog/2018/10/01/health-checking-grpc-servers-on-kubernetes/
[execprobe]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-a-liveness-command
[rel]: https://github.com/grpc-ecosystem/grpc-health-probe/releases
