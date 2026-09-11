# kafka-connect-custom-adaptors

A lightweight, high-performance Go microservice providing the runtime foundation for custom Kafka Connect adaptor workloads. Built with the Go standard library, containerized with Docker, and licensed under the VisionQuantech Custom Commercial License.

[![Go Version](https://img.shields.io/badge/Go-1.20-00ADD8?logo=go)](https://go.dev/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-VisionQuantech%20Custom-blueviolet)](./LICENSE)

---

## 📖 Overview

**kafka-connect-custom-adaptors** is a minimal, production-oriented Go service designed as the bootstrap runtime for custom Kafka Connect adaptor integrations. At its core, the application exposes a lightweight HTTP health/status endpoint that reports the operational state of the service, making it immediately suitable for containerized deployments, orchestration probes, and as a foundation for extending with custom Kafka Connect source/sink adaptor logic.

The service prioritizes:

- **Zero external dependencies** — built entirely on the Go standard library (`net/http`, `log`, `time`, `fmt`).
- **Fast startup & small footprint** — a single static binary produced via `go build`.
- **Container-first deployment** — ships with a ready-to-use `Dockerfile`.
- **Security-conscious defaults** — `.gitignore` hard-excludes secrets, credentials, and environment files from version control.

---

## 🏗️ Architecture & How It Works

The repository is intentionally lean. Here is exactly what the code does:

### Application Entry Point (`main.go`)

The entire runtime lives in a single `main` package:

1. **HTTP Handler Registration**
   ```go
   http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
       fmt.Fprintf(w, "System Operational: %s", time.Now())
   })
   ```
   A handler is registered on the root path (`/`) using Go's default `ServeMux`. Every incoming HTTP request receives a plain-text response containing the string `System Operational:` followed by the server's current timestamp (`time.Now()`). This doubles as a **liveness/health probe** — a `200 OK` response with a fresh timestamp confirms the service is alive and serving traffic.

2. **Server Startup**
   ```go
   log.Println("Starting high-performance service on :8080")
   log.Fatal(http.ListenAndServe(":8080", nil))
   ```
   The service binds to **port `8080`** on all interfaces and begins serving requests via the default mux. `log.Fatal` wraps the server call so that any fatal bind/listen error (e.g., port already in use) is logged and terminates the process with a non-zero exit code — the correct behavior for container restart policies.

3. **Startup Logging** — A structured log line is emitted to stdout on boot, which integrates cleanly with Docker logs and log aggregators.

### Module Definition (`go.mod`)

- **Module path:** `github.com/Shivay00001/kafka-connect-custom-adaptors`
- **Go version:** `1.20`
- **External dependencies:** *None.* The module builds with the standard library alone, guaranteeing reproducible builds with no supply-chain risk from third-party packages.

### Container Build (`Dockerfile`)

The image is built from `golang:1.20-alpine` — a minimal Alpine-based Go toolchain image:

| Step | Instruction | Purpose |
|------|-------------|---------|
| 1 | `FROM golang:1.20-alpine` | Lightweight base image with the Go 1.20 toolchain |
| 2 | `WORKDIR /app` | Sets `/app` as the working directory |
| 3 | `COPY . .` | Copies the full repository into the image |
| 4 | `RUN go build -o app` | Compiles the module into a single binary named `app` |
| 5 | `CMD ["./app"]` | Runs the binary when the container starts |

> **Note:** The container listens on port `8080` internally. You must publish this port when running the container (see below).

### Request Flow

```
Client ──HTTP GET /──▶ Docker Container :8080
                          │
                          ▼
                    Go net/http ServeMux
                          │
                          ▼
              Handler writes "System Operational: <timestamp>"
                          │
                          ▼
Client ◀── 200 OK (text/plain) ──────────────
```

---

## 🐳 Running with Docker

The application runs identically on any laptop or server with Docker installed.

### Option 1 — Standard Dockerfile (Recommended)

```bash
# 1. Clone the repository
git clone https://github.com/Shivay00001/kafka-connect-custom-adaptors.git
cd kafka-connect-custom-adaptors

# 2. Build the image
docker build -t kafka-connect-custom-adaptors .

# 3. Run the container, publishing port 8080
docker run -d --name kafka-adaptors -p 8080:8080 kafka-connect-custom-adaptors

# 4. Verify the service is operational
curl http://localhost:8080
# Expected output: System Operational: 2026-01-01 12:00:00 +0000 UTC ...
```

### Option 2 — Docker Compose

If you prefer Compose, create a `docker-compose.yml` in the repository root:

```yaml
services:
  app:
    build: .
    container_name: kafka-adaptors
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Then start it with:

```bash
docker-compose up -d --build
```

### Stopping & Cleanup

```bash
docker stop kafka-adaptors && docker rm kafka-adaptors
# or, with Compose:
docker-compose down
```

---

## 🛠️ Running Locally (Without Docker)

Requires **Go 1.20+**:

```bash
# Run directly
go run main.go

# Or build a binary and execute it
go build -o app
./app
```

Then visit `http://localhost:8080` in a browser or via `curl`.

---

## 📁 Repository Structure

```
kafka-connect-custom-adaptors/
├── Dockerfile      # Container build definition (golang:1.20-alpine)
├── go.mod          # Go module definition (no external dependencies)
├── main.go         # Application entry point — HTTP health service on :8080
├── LICENSE         # VisionQuantech Custom Commercial License
├── .gitignore      # Excludes secrets, env files, build artifacts
└── README.md       # This file
```

---

## 🔌 Extending with Custom Adaptors

This service is the intended foundation for custom Kafka Connect adaptor logic. Natural extension points include:

- Adding adaptor-specific HTTP routes alongside the `/` health endpoint.
- Registering source/sink connector configuration endpoints.
- Wiring the health handler into Kafka Connect worker readiness probes.

Contributions should preserve the zero-dependency philosophy where possible and keep secrets out of version control (see `.gitignore`).

---

## 📄 License

This project is distributed under the **VisionQuantech Custom Commercial License**:

- ✅ **Free** for personal, educational, non-financial, and non-earning use.
- 💰 **Revenue share (15–30%)** required for individuals earning revenue from this software.
- 🏢 **Commercial license required** for business/enterprise use — contact **visionquantech@proton.me**.

See [LICENSE](./LICENSE) for full terms. The software is provided **"AS IS"**, without warranty of any kind.

---

<p align="center">Copyright © 2026 Shivay00001 / VisionQuantech</p>