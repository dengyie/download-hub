<!-- markdownlint-disable-file MD033 MD045 -->
<div align="center">

<img src="brand/logo.png" alt="DownloadHub Logo" width="100" height="100" />

# DownloadHub

**High-Performance Digital Asset Distribution Platform, Zero-Dependency Storefront & Card-Key Verification Gateway**

[![License](https://img.shields.io/badge/license-MIT-green.svg?style=flat-square)](LICENSE)
[![Go Version](https://img.shields.io/badge/Go-1.26+-00ADD8.svg?style=flat-square&logo=go&logoColor=white)](https://golang.org)
[![Storage](https://img.shields.io/badge/Storage-OpenList_%7C_S3-blue.svg?style=flat-square)](https://github.com/OpenListTeam/OpenList)
[![Frontend](https://img.shields.io/badge/Frontend-Zero_Runtime_Deps-brightgreen.svg?style=flat-square)](frontend/)
[![Architecture](https://img.shields.io/badge/Architecture-Decoupled_SPA_%2B_Go_API-orange.svg?style=flat-square)](docs/frontend-architecture.md)
[![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg?style=flat-square)]()

[English](./README.md) · [简体中文](./README.zh-CN.md) · [Core Features](#core-features) · [Architecture](#system-architecture) · [Quick Start](#quick-start) · [API Endpoints](#api-endpoints) · [Configuration](#configuration-reference)

<br>

<p align="center">
  <b>Terminates server bandwidth saturation, unauthorized hotlinking, and quota double-spending during digital asset delivery.</b><br>
  Engineered with an ultra-lightweight Vanilla JS storefront, SQLite IMMEDIATE transactional leases, and dynamic short-lived signed URLs offloading to OpenList or S3 storage CDNs.<br>
  Delivers zero-npm client-side rendering, two-phase card key redemptions, idempotent grant recovery, zero-downtime hot backups, and comprehensive anti-tampering path protection.
</p>

</div>

---

## Motivation & Architecture Comparison

Independent creators, software engineers, and digital asset publishers often struggle with file delivery dilemmas: third-party commercial file hosts plague users with pop-up ads and artificial speed throttles, while self-hosted download portals frequently suffer from server bandwidth exhaustion, hotlinking abuse, and concurrent card-key redemption overdrafts:

| Evaluation Dimension | DownloadHub Platform | Traditional File Host Scripts | Commercial Card & Commerce SaaS |
| :--- | :--- | :--- | :--- |
| **Server Bandwidth** | Zero transit bandwidth; server only handles JSON metadata & signatures; files stream from CDN | All traffic routes through the server, rapidly saturating uplink bandwidth | Heavy proxy relays or forces proprietary download clients |
| **Concurrency & Integrity** | SQLite `IMMEDIATE` transaction leases; strictly prevents double-spending | Naive read-modify-write prone to race conditions and quota overdrafts | Heavy distributed locks requiring external Redis and complex setups |
| **Frontend Architecture** | Native Vanilla ES Modules; zero npm runtime dependencies; instant rendering | Bloated JavaScript bundles with slow load times and bundle churn | Cluttered legacy admin panels lacking mobile responsiveness |
| **Storage Security** | Generates temporary signed URLs with strict TTLs; storage credentials stay hidden | Exposes static S3 URLs or long-lived API tokens vulnerable to theft | Static links easily forwarded and shared publicly |
| **Backup & Management** | Zero-downtime hot backup via SQLite `VACUUM INTO`; native CLI tooling | Requires taking the service offline to prevent database snapshot corruption | Complex database backup orchestration pipelines |

---

## Core Features

| Module | Feature Highlight | Technical Implementation & Advantage |
| :--- | :--- | :--- |
| **Zero-Dependency UI** | Native Vanilla ES Modules | Zero client-side npm dependencies; instant loading with seamless light, dark, and system theme synchronization; Tabler vector icons. |
| **Resource Browsing** | Category filters & debounced search | Instant debounced filtering; strict whitelist-based HTML sanitization preventing stored XSS injection attacks in markdown descriptions. |
| **Two-Phase Redemption** | Strict transactional integrity | Phase 1 validates the card key and issues an in-memory lease token without deducting quota; Phase 2 deducts quota only when download is confirmed. |
| **Idempotent Recovery** | Resilient against network drops | Transient network drops or browser refreshes can safely recover the signed link using the original grant token without deducting quota twice. |
| **Flexible Key Models** | Path prefix or individual files | Grants access to specific files (`--resource`) or entire folder hierarchies (`--prefix`); supports finite counts or lifetime unlimited access. |
| **CDN Offloading** | Origin masking & direct delivery | Bridges to OpenList or standard S3 storage; offloads multi-gigabyte payloads directly to cloud CDNs, consuming zero gateway transit bandwidth. |
| **Adaptive Rate Limiting**| Fixed-window brute-force guard | In-process rate limiter per client IP mitigates automated card-key scanning; validates path canonicalization to reject directory traversal. |
| **Hot Backup & CLI** | Native administrative toolchain | Comprehensive CLI tools to issue, list, revoke keys, and export globally consistent hot backups via SQLite `VACUUM INTO` without downtime. |

---

## System Architecture

```text
       ┌────────────────────────┐
       │   Browser / Visitors   │
       └───────────┬────────────┘
                   │ HTTPS Request
                   ▼
       ┌────────────────────────┐
       │      Nginx Ingress     │ ◄─── Static Assets (Vanilla ES Modules SPA Shell)
       └───────────┬────────────┘
                   ├─────────────────────────────────────────┐
                   │ /api/* (Lightweight JSON API)           │ /p/* (Signed Large Files)
                   ▼                                         ▼
   ┌────────────────────────────────┐        ┌────────────────────────────────┐
   │       DownloadHub Server       │        │    OpenList / S3 Storage CDN   │
   │       (Go Engine :8080)        │───────►│    (Direct High-Speed Streams) │
   └───────────────┬────────────────┘ Sign   └────────────────────────────────┘
                   │                  Temporary Credentials   ▲
                   │ Strict IMMEDIATE                         │ Validate Signed
                   │ Atomic Transactions                      │ Token
                   ▼                                          │
   ┌────────────────────────────────┐                         │
   │       cards.db (SQLite)        │─────────────────────────┘
   │   (Cards, Quotas, Audit Log)   │
   └────────────────────────────────┘
```

---

## Quick Start

### 1. Build from Source

Prerequisites: **Go 1.26+** and **Node.js 22+** (required only for building the frontend):

```bash
# Clone the repository
git clone https://github.com/dengyie/DownloadHub.git
cd DownloadHub

# Compile the standalone backend binary
make build
# Output artifact: ./downloadhub

# Build frontend static bundle (outputs to frontend/dist)
npm --prefix frontend ci --ignore-scripts
npm --prefix frontend run build
```

To cross-compile for a Linux server target:

```bash
make build-linux
# Output artifact: ./downloadhub-linux-amd64
```

### 2. Configuration Setup

Copy the template configuration file:

```bash
cp config.yaml.example config.yaml
```

Configure `config.yaml` with your parameters (`openlist_admin_pass` and `openlist_min_url_ttl_seconds` are mandatory; setting TTL to `0` intentionally rejects startup):

```yaml
listen: "127.0.0.1:8080"
db_path: "./cards.db"
openlist_url: "http://127.0.0.1:5244"
openlist_admin_user: "admin"
openlist_admin_pass: "your_password"   # Required: OpenList administrator password

# Token validity window after redemption (seconds)
grant_window_seconds: 600

# Required: Minimum remaining validity for signed external URLs (seconds)
# Set to >= grant_window_seconds and <= 86400; setting 0 refuses launch
openlist_min_url_ttl_seconds: 3600

# Permitted download hostnames
download_hosts: ["download.example.com"]

# Trusted reverse proxy CIDRs for extracting client IP
trusted_proxy_cidrs: ["127.0.0.1/32"]
```

### 3. Key Generation & Management CLI

```bash
# Generate key: 3 download quotas, bound to private/release-v1.zip, valid for 30 days
./downloadhub gencard --quota 3 --resource private/release-v1.zip --days 30 --db cards.db

# Generate lifetime multi-resource key (--days 0 indicates lifetime validity)
./downloadhub gencard --quota 10 --resource private/doc.pdf,private/setup.exe --days 0 --db cards.db

# List all issued keys and quota balances
./downloadhub listcards --db cards.db

# Revoke a specific card key
./downloadhub revokecard --card <CARD_ID> --db cards.db

# Online zero-downtime backup (consistent SQLite snapshot)
./downloadhub backup --db cards.db --out cards-snapshot.db
```

### 4. Running the Service

```bash
./downloadhub serve --config config.yaml
```

For production reverse-proxy deployments using Nginx to serve the SPA and route API calls, refer to [deploy/download-nginx.conf.example](deploy/download-nginx.conf.example).

---

## API Endpoints

| Method | Endpoint | Description | Consumes Quota |
| :--- | :--- | :--- | :--- |
| `GET` | `/`, `/redeem`, `/orders` | SPA frontend route navigation | No |
| `GET` | `/api/catalog` | Whitelist catalog of publicly downloadable assets | No |
| `POST` | `/api/redeem` | Validates card key and returns grant lease token | No |
| `POST` | `/api/grants` | Submits grant lease and returns signed download URL | Yes (atomically deducts 1) |
| `GET` | `/health` | Liveness health check endpoint | No |
| `GET` | `/ready` | Database readiness probe | No |

---

## Custom Site Branding

Storefront branding labels are centralized in [frontend/src/site.js](frontend/src/site.js) (site title, navigation marks, and hero taglines). Combine with default values in `frontend/index.html` and assets in the `brand/` directory to rebrand the interface without altering application logic.

---

## Testing & Quality Gates

The repository includes test suites covering high-concurrency race conditions and boundary validations:

```bash
# Run complete test suite (requires Go, Node 22+, Python 3, PHP 8.2+ with SQLite, and Playwright)
make check

# Run Go backend unit tests with race detector
go test -race -count=1 ./...

# Run frontend tests and validation
make check-frontend
```

---

## Contributing & License

Contributions are welcome. Please ensure all tests pass prior to submitting changes:

```bash
go test -v ./...
```

- Contribution Guidelines: [CONTRIBUTING.md](CONTRIBUTING.md)
- Security Policy: [SECURITY.md](SECURITY.md)
- Licensed under the [MIT License](LICENSE).
