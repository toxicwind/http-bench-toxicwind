# PR: Sandbox K8s Benchmark Results + websockets v15 Compatibility

## Branch: `feat/sandbox-benchmark-results`

### Changes
- [x] Add `results/sandbox_k8s_2026-08-18.json` — benchmark data for 5 HTTP clients
- [x] Add `results/README.md` — analysis and findings
- [x] Document curl_cffi async = 7x faster than requests
- [x] Document proxy overhead (~500ms per request)
- [x] Document websockets v15 CancelledError fix
- [x] Document CDP as alternative for blocked sites

### Benchmark Environment
- K8s sandbox with s6 init
- Chrome proxy: `10.86.13.73:5900`
- DNS: `192.168.0.10` (K8s) + `8.8.8.8`
- Python 3.12, 20 requests per client

### Results Summary

| Client | Per Request | vs Baseline |
|--------|-------------|-------------|
| **curl_cffi async** | **40.6ms** | **7x faster** |
| requests | 279.2ms | 1.0x |
| httpx | 289.2ms | 0.96x |

### Key Finding
In containerized environments with proxy routing, `curl_cffi` async is the clear winner due to connection reuse and TLS handshake batching. The proxy adds ~500ms overhead, making it critical to minimize connection count.

### No Breaking Changes
All additions are additive. Existing benchmark code unchanged.
