
# OpenContainer

**Lightweight, rootless “containers” for ANY device**  
A drop‑in, Docker‑compatible orchestration engine built on PRoot.  
Run full multi‑service stacks (PostgreSQL, Node.js, Redis, Supabase, and more) on unrooted Android, iOS, Linux, macOS, Windows (via WSL/QEMU), and other constrained environments.

---

## 🚀 Project Overview

OpenContainer is an open‑source, cross‑platform containerization system that lets you:

- **Pull & run real Docker images** (Alpine, Ubuntu, Postgres, Node, etc.)  
- **Compose multi‑service stacks** via `docker-compose.yml`–style files  
- **Isolate each service** in its own PRoot‑based rootfs (no kernel namespaces or cgroups needed)  
- **Expose ports & mount volumes** just like Docker, on `localhost`  
- **Stay entirely unprivileged**—no root, no VM, no kernel modules  

---

## 🎯 Motivation

1. **No Root, No VMs**  
   Android devices with broken UIs, iOS sandboxes, school computers, WSL… all can run full container stacks without hacking the kernel or paying the VM penalty.  
2. **Docker Compatibility**  
   Leverage the vast Docker Hub ecosystem. Pull official images, parse their layers, honor `ENTRYPOINT`/`CMD`/`ENV`, and wire them together via Compose.  
3. **Lightweight & Portable**  
   Skip `dockerd`, containerd, runc, systemd, and all that bloat. Your total footprint is a single lightweight binary (Rust/C++) plus a handful of rootfs folders.

---

## ✨ Key Features

- **OCI Image Support**  
  - Pull from Docker Hub or private registries  
  - Unpack layers, apply whiteouts, reconstruct rootfs  
  - Read `Config` (CMD, Entrypoint, Env, Volumes)

- **Compose Orchestration**  
  - Parse `docker-compose.yml` (services, ports, volumes, networks, dependencies)  
  - Auto‑generate PRoot startup scripts (`run.sh`) for each service  
  - Single command: `opencontainer up`, `opencontainer down`, `opencontainer ps`, `opencontainer logs`

- **PRoot Isolation**  
  - Per‑service `proot -R ./rootfs` with bind mounts, cwd, user‑0 emulation  
  - Shared network via localhost port mapping  
  - Volume support via `-b /host/path:/container/path`

- **Cross‑Platform**  
  - Android (Termux, UserLAnd), iOS (iSH‑style/WASM), Linux, macOS, Windows (WSL/QEMU)  
  - Single codebase in Rust (or C++)—compile to native or to WASM for special targets

- **Extensible CLI**  
  - Plugin‑friendly: custom commands, hooks, healthchecks  
  - TUI dashboard (future)

---

## 📐 Architecture

+––––––––––––––––––––––––––––+
| ● opencontainer (Rust/C++)                             |
|    ├─ CLI parser (Compose YAML → AST)                  |
|    ├─ Image manager (skopeo/registry client)           |
|    ├─ Layer unpacker & rootfs builder                  |
|    ├─ Orchestrator (dependency graph, ordering)        |
|    ├─ Runner → generates per‑service run.sh scripts    |
|    └─ Controller (up/down/ps/logs)                     |
+––––––––––––––––––––––––––––+
│                   │            │
▼                   ▼            ▼
./.containers/  ./.containers/
├─ rootfs/             ├─ rootfs/
├─ run.sh              ├─ run.sh
└─ volumes/            └─ volumes/

- **Compose Parser**  
  Uses `serde_yaml` to translate your Compose file into an in‑memory model.

- **Image Manager**  
  Pulls & caches images; unpacks layers in correct order, applies metadata.

- **Orchestrator & Runner**  
  Walks the dependency graph (`depends_on`), allocates ports, writes `run.sh` scripts invoking `proot`.

- **Controller**  
  Implements Docker‑style commands (`up`, `down`, `ps`, `logs`) by launching/stopping those scripts and tailing logs.

---

## 🛠️ Getting Started

```bash
# Clone & build
git clone https://github.com/AIRAS-The-Innovations-Of-New-Era/OpenContainer.git

cd OpenContainer
cargo build --release            # or cmake/make for C++

# Install the binary
cp target/release/opencontainer /usr/local/bin/

# Prepare a compose file
cat > docker-compose.yml <<EOF
services:
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=example
    volumes:
      - dbdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  api:
    image: node:18-alpine
    command: "npm start"
    depends_on:
      - postgres
    ports:
      - "3000:3000"

volumes:
  dbdata:
EOF

# Launch everything
opencontainer up
opencontainer ps
opencontainer logs api


⸻

📦 Roadmap
	•	v0.1: Core CLI, image pull & unpack, PRoot runner, basic Compose support
	•	v0.2: Volume snapshots, env interpolation, healthchecks, depends_on ordering
	•	v0.3: Networking proxy (virtual bridge), TUI dashboard, plugin system
	•	v1.0: Web UI (Portainer‑style), Windows native support, iOS WASM bundle

⸻

🤝 Contributing & License

OpenContainer is 100% open‑source under the Apache 2.0 License. We welcome:
	•	Issues & feature requests
	•	Pull Requests (Rust/C++/docs)
	•	Templates for common stacks (Supabase, Strapi, Appwrite, etc.)
	•	Platform‑specific tweaks (Android/iOS packaging)

See CONTRIBUTING.md and LICENSE for details.

⸻

© 2025 OpenContainer Contributors
“Containers for Everyone, Everywhere—No Root Required.”

