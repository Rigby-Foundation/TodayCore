# TodayCore extensions

This document covers configuration fields added by TodayCore on top of sing-box 1.15.0-alpha.6.

> [!IMPORTANT]
> The extensions described here are client/outbound features. TodayCore does not provide an XHTTP inbound or a VLESS Encryption server.

## XHTTP outbound transport

XHTTP is configured inside a compatible outbound's `transport` object. TodayCore uses Xray-compatible camelCase field names for XHTTP-specific options.

### Minimal VLESS + XHTTP example

```json
{
  "outbounds": [
    {
      "type": "vless",
      "tag": "vless-xhttp",
      "server": "server.example.com",
      "server_port": 443,
      "uuid": "00000000-0000-0000-0000-000000000000",
      "tls": {
        "enabled": true,
        "server_name": "server.example.com",
        "alpn": ["h2", "http/1.1"],
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        }
      },
      "transport": {
        "type": "xhttp",
        "host": "server.example.com",
        "path": "/api",
        "mode": "auto"
      }
    }
  ]
}
```

The server, UUID, host, path, TLS server name, and other authentication values must match the Xray server configuration.

### VLESS + XHTTP + REALITY example

```json
{
  "outbounds": [
    {
      "type": "vless",
      "tag": "vless-xhttp-reality",
      "server": "203.0.113.10",
      "server_port": 443,
      "uuid": "00000000-0000-0000-0000-000000000000",
      "tls": {
        "enabled": true,
        "server_name": "www.example.com",
        "alpn": ["h2", "http/1.1"],
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        },
        "reality": {
          "enabled": true,
          "public_key": "REPLACE_WITH_REALITY_PUBLIC_KEY",
          "short_id": "0123456789abcdef"
        }
      },
      "transport": {
        "type": "xhttp",
        "host": "www.example.com",
        "path": "/assets",
        "mode": "stream-up"
      }
    }
  ]
}
```

The `X25519MLKEM768` compatibility fix is applied automatically to supported static uTLS fingerprints. No TodayCore-specific REALITY field is required.

### Modes

| Value | Meaning |
| --- | --- |
| `auto` | Select behavior automatically; this is the default |
| `packet-up` | Send uplink data as independent HTTP requests |
| `stream-up` | Stream the uplink while keeping a separate downlink |
| `stream-one` | Use one streaming request/response pair |

### Advanced XHTTP example

Ranges accept either a number such as `5` or a string such as `"5-10"`.

```json
{
  "type": "xhttp",
  "host": "cdn.example.com",
  "path": "/api?ed=2560",
  "mode": "packet-up",
  "headers": {
    "User-Agent": "Mozilla/5.0",
    "Accept-Language": "en-US,en;q=0.9"
  },
  "xPaddingBytes": "100-1000",
  "xPaddingObfsMode": true,
  "xPaddingKey": "x_padding",
  "xPaddingHeader": "Referer",
  "xPaddingPlacement": "queryInHeader",
  "xPaddingMethod": "repeatX",
  "uplinkHTTPMethod": "POST",
  "sessionIDPlacement": "path",
  "seqPlacement": "path",
  "uplinkDataPlacement": "body",
  "uplinkChunkSize": "3000-4000",
  "scMaxEachPostBytes": 1000000,
  "scMinPostsIntervalMs": 30,
  "scMaxBufferedPosts": 30,
  "scStreamUpServerSecs": "20-80",
  "serverMaxHeaderBytes": 8192,
  "xmux": {
    "maxConnections": 3,
    "cMaxReuseTimes": "0-0",
    "hMaxRequestTimes": "600-900",
    "hMaxReusableSecs": "1800-3000",
    "hKeepAlivePeriod": 0
  }
}
```

### XHTTP field reference

| Field | Accepted values / purpose |
| --- | --- |
| `host` | HTTP Host/authority used by the XHTTP server |
| `path` | Request path; may include a query string |
| `mode` | `auto`, `packet-up`, `stream-up`, or `stream-one` |
| `headers` | Additional request headers; do not put `Host` here—use `host` |
| `xPaddingBytes` | X-Padding size or range; values must be positive when explicitly set |
| `xPaddingObfsMode` | Enable configurable padding placement and method |
| `xPaddingKey` | Padding query/cookie/header key |
| `xPaddingHeader` | Carrier header used by padding placement |
| `xPaddingPlacement` | `cookie`, `header`, `query`, or `queryInHeader` |
| `xPaddingMethod` | `repeatX` or `tokenish` |
| `uplinkHTTPMethod` | Usually `POST`; `GET` is valid only with `packet-up` |
| `sessionIDPlacement` | `path`, `cookie`, `header`, or `query` |
| `sessionIDKey` | Custom key when the session ID is not placed in the path |
| `sessionIDTable` | Custom ID character table or a predefined alias such as `Base62`, `hex`, or `number` |
| `sessionIDLength` | Generated session-ID length or range |
| `seqPlacement` | `path`, `cookie`, `header`, or `query` |
| `seqKey` | Custom sequence key when not placed in the path |
| `uplinkDataPlacement` | `auto`, `body`, `cookie`, or `header`; cookie/header require `packet-up` |
| `uplinkDataKey` | Custom key for uplink data outside the body |
| `uplinkChunkSize` | Uplink chunk size or range |
| `noGRPCHeader` | Do not add the gRPC-like content type on streaming uplinks |
| `noSSEHeader` | Disable the SSE-like response header behavior |
| `scMaxEachPostBytes` | Maximum bytes in each packet-up POST |
| `scMinPostsIntervalMs` | Minimum interval between packet-up posts |
| `scMaxBufferedPosts` | Maximum number of buffered packet-up requests |
| `scStreamUpServerSecs` | Stream-up server duration or range |
| `serverMaxHeaderBytes` | Maximum accepted HTTP response-header size |
| `xmux` | XHTTP connection and request reuse controls |
| `downloadSettings` | Optional separate server/TLS/XHTTP settings for the downlink |

`xmux.maxConnections` and `xmux.maxConcurrency` are mutually exclusive. If the entire `xmux` object is omitted, TodayCore follows the Xray defaults used by the port.

### Separate download settings

```json
{
  "type": "xhttp",
  "host": "upload.example.com",
  "path": "/up",
  "mode": "packet-up",
  "downloadSettings": {
    "server": "download.example.com",
    "server_port": 443,
    "tls": {
      "enabled": true,
      "server_name": "download.example.com",
      "alpn": ["h2"]
    },
    "host": "download.example.com",
    "path": "/down",
    "mode": "stream-one"
  }
}
```

### XHTTP limitations

- Outbound/client only; `transport.type: "xhttp"` on an inbound returns an error.
- HTTP/3 is not supported. Configure TLS ALPN as `h2`, `http/1.1`, or both.
- Xray's browser dialer is not included.
- Use an Xray-compatible server. TodayCore is not an XHTTP server.

## VLESS Encryption

VLESS Encryption is configured with the `encryption` field on a VLESS outbound.

### Example

```json
{
  "outbounds": [
    {
      "type": "vless",
      "tag": "vless-pq",
      "server": "server.example.com",
      "server_port": 443,
      "uuid": "00000000-0000-0000-0000-000000000000",
      "encryption": "mlkem768x25519plus.native.0rtt.100-111-1111.75-0-111.50-0-3333.REPLACE_WITH_BASE64URL_PUBLIC_KEY",
      "tls": {
        "enabled": true,
        "server_name": "server.example.com"
      }
    }
  ]
}
```

Use the complete encryption string generated for or copied from the matching Xray server. Do not copy the placeholder literally.

### String format

```text
mlkem768x25519plus.<xor-mode>.<rtt-mode>[.<padding>].<key>[.<relay-key>...]
```

| Part | Values |
| --- | --- |
| Algorithm | `mlkem768x25519plus` |
| XOR mode | `native`, `xorpub`, or `random` |
| RTT mode | `1rtt` or `0rtt` |
| Padding | Optional dot-separated `chance-min-max` elements |
| Key | Raw URL-safe base64 without padding; decoded length must be 32 or 1184 bytes |
| Relay keys | Optional additional keys in relay order |

An empty `encryption` value or `"none"` disables VLESS Encryption.

### VLESS Encryption limitations

- Client/outbound only.
- Experimental and not yet broadly interoperability-tested.
- The server must support the same Xray VLESS Encryption wire format.
- A wrong key, order, mode, or padding string prevents the handshake.
- Keep all private key material on the server. The client configuration uses public key material supplied by the server operator.

## Compatibility summary

TodayCore currently ports the XHTTP client and modern REALITY/VLESS client compatibility needed for current deployments. It does **not** port XHTTP inbound, HTTP/3 XHTTP, the Xray browser dialer, REALITY `mldsa65Verify`, `spiderX`, FinalMask, or a VLESS Encryption server.

For standard sing-box fields, consult the existing documentation under `docs/configuration/`.