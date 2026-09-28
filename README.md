[Русская версия](README.ru.md)

# TodayCore

**TodayCore** is an experimental networking core based on **sing-box 1.15.0-alpha.6**. It keeps the sing-box architecture and configuration model while adding selected, modern client-side features ported from **Xray-core 26.9.9**.

> [!WARNING]
> TodayCore is a beta, community-driven project. Do not expect a fixed release schedule, guaranteed compatibility, or production support. Test every update before deploying it.

## Why TodayCore exists

The project explores interoperability between sing-box and current Xray deployments without turning sing-box into a complete Xray clone. Only relevant modern features are considered; obsolete and legacy protocols are intentionally outside the project scope.

The current codebase is identified as **TodayCore**, but its upstream base is **sing-box 1.15.0-alpha.6**.

## Added on top of sing-box

### XHTTP outbound transport

The Xray SplitHTTP/XHTTP **client transport** was ported from Xray-core 26.9.9.

Implemented:

- `packet-up`, `stream-up`, `stream-one`, and `auto` modes;
- HTTP/1.1 and HTTP/2 operation;
- session metadata in path, query, header, or cookie;
- uplink data in body, header, or cookie;
- X-Padding, including custom placement, key, header, and obfuscation mode;
- browser-like default request headers;
- XMUX connection and request reuse settings;
- separate `downloadSettings` support;
- raw HTTP/1.1 keep-alive, upload queue, and split-connection behavior.

XHTTP is available only as an **outbound/client transport**.

### Modern REALITY compatibility

The REALITY client was updated for compatibility with Xray-core 26.9.9 servers:

- restores the `X25519MLKEM768` hybrid key share before plain `X25519`;
- keeps the X25519 key share needed by REALITY authentication;
- reports protocol-compatible client version `26.9.9` in the REALITY session ID;
- avoids the common fallback-to-destination failure that appears as `reality verification failed` against servers requiring modern clients.

This behavior is automatic when using REALITY with uTLS.

### VLESS post-quantum encryption

TodayCore contains an experimental client-side port of Xray's VLESS Encryption:

- `mlkem768x25519plus`;
- `native`, `xorpub`, and `random` modes;
- `1rtt` and `0rtt` handshakes;
- padding profiles;
- 32-byte X25519 and 1184-byte ML-KEM-768 public keys;
- relay key chains.

The implementation is currently **client/outbound only** and should be treated as experimental until it receives broader interoperability testing.

## Porting status

| Xray feature | Status in TodayCore | Notes |
| --- | --- | --- |
| XHTTP outbound/client | Ported | HTTP/1.1 and HTTP/2 |
| XHTTP `packet-up` | Ported | Client side |
| XHTTP `stream-up` | Ported | Client side |
| XHTTP `stream-one` | Ported | Client side |
| XHTTP metadata placement and X-Padding | Ported | Xray-compatible field names |
| XHTTP XMUX | Ported | Connection/request reuse controls |
| REALITY `X25519MLKEM768` compatibility | Ported | Automatic with supported uTLS fingerprints |
| VLESS `mlkem768x25519plus` encryption | Experimental | Client/outbound only |
| XHTTP inbound/server | Not ported | Explicitly rejected by the configuration runtime |
| XHTTP over HTTP/3 | Not ported | Use `h2` or `http/1.1`; the projects use incompatible QUIC forks |
| Xray browser dialer | Not ported | No browser-dialer integration |
| REALITY `mldsa65Verify` | Not ported | Not implemented |
| REALITY `spiderX` | Not ported | Not implemented |
| FinalMask | Not ported | Not implemented |
| VLESS Encryption inbound/server | Not ported | Client-only implementation |

## Configuration

TodayCore continues to use the sing-box JSON configuration format. Documentation and examples for TodayCore-specific additions are available in:

- [TodayCore extensions and configuration examples](docs/todaycore-extensions.md)

The original sing-box documentation remains in [`docs/`](docs/). For fields not described in the TodayCore extension guide, follow the upstream sing-box 1.15 configuration documentation.

## Build

### Requirements

- Go **1.25.5** or a compatible newer toolchain;
- Git;
- `make` for the standard build workflow.

### Standard build

```bash
make
```

Install to `$GOBIN`:

```bash
make install
```

A custom build can be requested through the Makefile:

```bash
TAGS="with_quic with_utls" make
```

The default build tags and required linker flags are maintained in `release/DEFAULT_BUILD_TAGS*` and `release/LDFLAGS`. Prefer the standard `make` target unless you know which platform-specific features you need.

## Development status and contributions

TodayCore is maintained **entirely on a voluntary basis**. Development may pause for long periods, and there is no promise that upstream sing-box or Xray changes will be merged immediately.

If you want the project to stay active, the best way to help is to participate directly:

1. test TodayCore against current Xray servers;
2. report reproducible compatibility problems;
3. include logs, server/client versions, and a sanitized configuration;
4. add tests for protocol and wire-format behavior;
5. open a pull request with a focused, reviewable change.

Useful commits and pull requests are the strongest signal that there is real interest in continued development. Community initiative is welcome, and external contributions can directly determine how quickly the project moves forward.

Please keep pull requests small where possible, explain the upstream behavior being matched, and link to the corresponding Xray implementation or specification.

## Upstream projects and attribution

TodayCore is derived from:

- [SagerNet/sing-box](https://github.com/SagerNet/sing-box), base version 1.15.0-alpha.6;
- [XTLS/Xray-core](https://github.com/XTLS/Xray-core), source reference version 26.9.9 for selected ports.

XHTTP-derived files retain their Xray/MPL attribution. TodayCore modifications remain subject to the repository's GPL-3.0-or-later licensing terms. Review source-file headers and [`LICENSE`](LICENSE) before redistribution.

TodayCore is an independent community project and is not an official SagerNet or XTLS release. The TodayCore name must not be used to imply endorsement by either upstream project.

## Security

This project handles low-level networking, cryptography, and experimental protocol compatibility. Never assume that a successful build has been security-audited. Avoid publishing private keys, UUIDs, REALITY keys, or complete production configurations in bug reports.