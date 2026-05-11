# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cinatra is a high-performance, header-only C++20 HTTP framework supporting HTTP/1.1, HTTPS, WebSocket, SSE, reverse proxy, and AOP. It uses C++20 coroutines (via async_simple) and ASIO for async I/O.

## Build Commands

```bash
# Configure (from a build directory)
cmake ..
cmake -DCMAKE_BUILD_TYPE=Debug ..        # Debug with ASAN
cmake -DENABLE_SSL=ON ..                  # Enable HTTPS
cmake -DENABLE_SIMD=AVX2 ..              # SIMD (SSE42/AVX2/AARCH64, pick one)
cmake -DBUILD_UNIT_TESTS=OFF ..          # Skip tests
cmake -DCOVERAGE_TEST=ON ..              # Coverage

# Build
make -j$(nproc)

# Run all tests
ctest

# Run specific test binary
./tests/test_cinatra
./tests/test_http_parse
./tests/test_corofile
./tests/test_time_util
./tests/test_rlimiter
./tests/test_metric
```

## Key CMake Options

| Option | Default | Description |
|--------|---------|-------------|
| `BUILD_UNIT_TESTS` | ON | Build unit tests (doctest) |
| `BUILD_EXAMPLES` | ON | Build examples |
| `BUILD_PRESS_TOOL` | ON | Build HTTP benchmark tool |
| `ENABLE_SSL` | OFF (ON on macOS) | HTTPS via OpenSSL |
| `ENABLE_SIMD` | OFF | SSE42, AVX2, or AARCH64 |
| `ENABLE_SANITIZER` | ON | AddressSanitizer in Debug |
| `ENABLE_METRIC_JSON` | OFF | Serialize metrics to JSON (requires iguana) |
| `COVERAGE_TEST` | OFF | Code coverage flags |

## Architecture

### Header-Only Library

All framework code is in `include/cinatra/`. The single public entry point is `include/cinatra.hpp`.

### Dependencies (fetched via CMake FetchContent)

- **ASIO** (1.38.0) — async network I/O, standalone mode (`ASIO_STANDALONE`)
- **async_simple** (v1.3) — Alibaba's C++20 coroutine library (`UTHREAD=OFF`)
- **iguana** — JSON serialization (git submodule, optional; required for `ENABLE_METRIC_JSON`)

Optional system deps: OpenSSL, ZLIB, Brotli.

### Core Components

- **`coro_http_server.hpp`** — Main server. Uses `io_service_pool` (thread-per-io_context). Handlers registered via `set_http_handler<GET, POST>(path, handler)`.
- **`coro_http_client.hpp`** — Async HTTP client with connection pooling, redirects, proxy, WebSocket.
- **`coro_http_connection.hpp`** — Per-connection coroutine managing read/write loops. Parses HTTP with `picohttpparser`.
- **`coro_http_request.hpp`** / **`coro_http_response.hpp`** — Request/response objects with headers, body, cookies, multipart, session.
- **`coro_http_router.hpp`** + **`coro_radix_tree.hpp`** — Radix-tree-based URL routing with path parameters and AOP (before/after hooks).
- **`http_parser.hpp`** — Wraps picohttpparser for HTTP message parsing.
- **`websocket.hpp`** — WebSocket frame handling (builds on coro_http_connection).
- **`session_manager.hpp`** / **`session.hpp`** — Server-side session storage (memory-backed).
- **`multipart.hpp`** — Multipart form/file upload parsing.
- **`rate_limiter.hpp`** — Token bucket rate limiter.
- **`cinatra_log_wrapper.hpp`** — Logging facade (uses `spdlog` if available).

### Data Flow

1. `coro_http_server` accepts connections on a thread pool of `io_context`s
2. Each connection spawns a coroutine in `coro_http_connection`
3. Request is parsed (`http_parser`), routed (`coro_http_router`/`coro_radix_tree`), and dispatched to the registered handler
4. Handler writes to `coro_http_response`, which serializes and sends the response

### Test Framework

Tests use **doctest** (single header). Test macros: `TEST_CASE`, `CHECK`, `REQUIRE`. Tests define `INJECT_FOR_HTTP_CLIENT_TEST` and `INJECT_FOR_HTTP_SEVER_TEST` to expose internals.

### Compiler Requirements

- GCC 10.2+ needs `-fcoroutines` and `-fno-tree-slp-vectorize` (Release)
- Clang 13+ works with `-std=c++20`
- MSVC needs `/std:c++latest`, `/bigobj`, `/EHa`

### Code Style

CI enforces **clang-format** (Google style, BraceWrapping: `BeforeElse`, `BeforeCatch`). See `.clang-format` for details.
