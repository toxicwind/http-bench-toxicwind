# Sandbox Benchmark Results

## K8s Container Environment (2026-08-18)

### Setup
- **Container:** K8s pod with s6 init system
- **Proxy:** Chrome routes through `10.86.13.73:5900`
- **DNS:** K8s cluster DNS (`192.168.0.10`) + Google/Cloudflare fallbacks
- **Python:** 3.12
- **Requests:** 20 per client

### Results

| Client | Total (20 reqs) | Per Request | vs Baseline |
|--------|----------------|-------------|-------------|
| **curl_cffi async** | **811ms** | **40.6ms** | **7x faster** |
| requests | 5584ms | 279.2ms | 1.0x |
| curl_cffi sync | 5739ms | 287.0ms | 0.97x |
| httpx | 5784ms | 289.2ms | 0.96x |
| curl_cffi + proxy | 15902ms | 795.1ms | 0.35x |

### Key Findings

1. **curl_cffi async is 7x faster** than requests/httpx in K8s sandboxes
2. **Proxy adds ~500ms overhead** per request (connection establishment through proxy)
3. **Connection reuse is critical** — async reuses curl handles, sync creates new ones
4. **TLS impersonation has overhead** — curl_cffi sync is slightly slower than requests

### websockets v15 Compatibility Issue

When using `asyncio.wait_for(ws.recv(), timeout=X)` with websockets 15.0+:
- **BROKEN:** Causes CancelledError cascade
- **FIX:** Use `asyncio.timeout()` context manager instead

```python
# BROKEN
msg = await asyncio.wait_for(ws.recv(), timeout=5)

# FIXED
async with asyncio.timeout(5):
    msg = await ws.recv()
```

### CDP as Alternative

For sites that block HTTP clients (Cloudflare, DataDome, Kasada):
- Use Chrome DevTools Protocol via `ws://127.0.0.1:9223`
- Inherits Chrome's proxy configuration automatically
- ~3-5s per page (faster than Playwright's ~8-12s)
