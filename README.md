# 🔷 Arc Node — Local Testnet Setup Guide

> Run a local copy of the [Arc Network](https://www.arc.network/) — Circle's stablecoin-native L1 blockchain — on your own machine using Docker. Test, explore, and hunt bugs without touching the public testnet.

[![Arc Testnet](https://img.shields.io/badge/Arc-Testnet-blue?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48Y2lyY2xlIGN4PSIxMiIgY3k9IjEyIiByPSIxMCIgZmlsbD0iIzAwNzBGMyIvPjwvc3ZnPg==)](https://testnet.arcscan.app)
[![Docker](https://img.shields.io/badge/Docker-required-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docs.docker.com/get-docker/)
[![Node.js](https://img.shields.io/badge/Node.js-v22+-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![Rust](https://img.shields.io/badge/Rust-required-CE422B?style=flat-square&logo=rust&logoColor=white)](https://rustup.rs)
[![HackerOne](https://img.shields.io/badge/Bug%20Bounty-HackerOne-25282B?style=flat-square&logo=hackerone)](https://hackerone.com/circle-bbp)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## 📌 Local Testnet vs Public Testnet

Before you begin, understand the difference:

| | **Local Testnet** *(this guide)* | **Public Testnet Node** |
|---|---|---|
| Connected to internet | ❌ No | ✅ Yes |
| Visible to others | ❌ No | ✅ Yes |
| Hardware requirements | Low–Medium | 64 GB RAM + 1 TB NVMe |
| Use case | Testing, bug hunting, development | Running a real network node |
| Setup time | 30–60 min (first run) | Hours (snapshot download) |

This guide covers the **Local Testnet** path using Docker and `make testnet`. You get 5 validator nodes + 1 full node + a block explorer running entirely on your machine.

---

## 🐛 What Is the Bug Bounty?

Arc rewards researchers who find and report vulnerabilities through [HackerOne](https://hackerone.com/circle-bbp).

- **Reward range:** $150 – $5,000+
- **Campaign period:** April 9 – June 1, 2026
- **Scope:** Local testnet only — do **not** test against the public testnet
- **Goal:** Find bugs that affect network safety, liveness, correctness, or reliability

> Set up the node → run it → try to break or slow down the system → report reproducible findings on HackerOne.

---

## ⚙️ Prerequisites

- Linux (Ubuntu 22.04+ or Debian 12+ recommended)
- `sudo` access
- A stable internet connection for the initial setup

---

## 🚀 Installation

### Step 1 — Install Dependencies

> ⚠️ **If you already have Docker CE** (from `download.docker.com`), skip `docker.io` — they conflict. If you already have Node.js v22 from NodeSource, skip `npm` — it's bundled.

```bash
sudo apt-get update
sudo apt-get install git make libclang-dev curl -y
```

Verify Node.js (should be v22+):

```bash
node -v   # Expected: v22.x.x
npm -v    # Bundled with NodeSource Node.js
```

---

### Step 2 — Setup Docker

```bash
sudo service docker start
sudo usermod -aG docker $USER
```

> 👉 **Reconnect your terminal / SSH session after this step** for the group change to take effect.

Verify:

```bash
docker --version
docker compose version
```

---

### Step 3 — Clone the Repository

```bash
cd ~
git clone https://github.com/circlefin/arc-node
cd arc-node
git submodule update --init --recursive
```

---

### Step 4 — Install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup -i v1.4.4
```

Verify:

```bash
cast --version
```

---

### Step 5 — Update Docker Compose (if needed)

```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-linux-x86_64 \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
docker compose version
```

---

### Step 6 — Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# Press 1 when prompted
source $HOME/.cargo/env
```

Verify:

```bash
rustc --version
cargo --version
```

---

### Step 7 — Install Project Dependencies

```bash
cd ~/arc-node
npm install
```

---

### Step 8 — Run the Arc Node

```bash
cd ~/arc-node
make testnet
```

> ⏳ First run takes **30–60 minutes** — Docker pulls images and builds the network. This is normal.

---

## ✅ Success Output

When everything is running, you'll see:

```
Testnet started
- Prometheus:     http://localhost:9090
- Grafana:        http://localhost:3000
- Block explorer: http://localhost:80
```

Open **http://localhost** in your browser. The Arc Localnet Explorer will load and you'll see blocks being produced in real time.

---

## 🧪 Send Test Transactions

```bash
make testnet-load RATE=10 TIME=30
```

This sends **10 transactions per second for 30 seconds**. Watch them appear in the Explorer live.

> 📊 Example results: 500+ transactions, 4,800+ blocks produced, gas fee = **0.00003 USDC**

---

## 🛑 Stop / Clean Up

Stop the testnet:

```bash
make testnet-down
```

Remove everything (containers, volumes, data):

```bash
make testnet-clean
```

---

## 🔧 Common Errors & Fixes

### ❌ `permission denied while trying to connect to Docker`

Docker group permission is missing.

```bash
sudo usermod -aG docker $USER
# Then close and reopen the terminal
```

---

### ❌ `unknown shorthand flag: 'f' in -f`

Docker Compose version is outdated. Redo [Step 5](#step-5--update-docker-compose-if-needed).

---

### ❌ `Pool overlaps with other one on this address space`

Old Docker networks are conflicting.

```bash
# Quick fix
docker network prune -f

# If that doesn't work, remove named networks manually
docker network ls
docker network rm arc_testnet_default arc_testnet_host-access arc_testnet_monitoring_default
```

If it still fails, change Docker's default subnet:

```bash
sudo nano /etc/docker/daemon.json
```

Paste:

```json
{
  "default-address-pools": [
    {"base": "192.168.0.0/16", "size": 24}
  ]
}
```

Save (`Ctrl+X` → `Y` → `Enter`), then restart Docker:

```bash
sudo service docker restart
```

---

### ❌ `bind source path does not exist: .../blockscout/logs`

Required folders are missing.

```bash
mkdir -p ~/arc-node/.quake/localdev/blockscout/logs
mkdir -p ~/arc-node/.quake/localdev/blockscout/dets
```

---

### ❌ `Conflict. The container name is already in use`

Old containers are still running.

```bash
make testnet-down
make testnet-clean
make testnet
```

---

### ❌ Explorer shows "No data" and doesn't update

Permission issue with Blockscout's data folder.

```bash
chmod -R 777 ~/arc-node/.quake/localdev/blockscout/
docker restart backend
```

Refresh the browser page.

---

### ❌ `containerd.io : Conflicts: containerd` (APT error)

You have Docker CE installed from `download.docker.com`. Do **not** install `docker.io` — it conflicts. Use the Docker CE you already have:

```bash
sudo apt-get install docker-ce docker-ce-cli -y
```

---

### ❌ `npm : Depends: node-agent-base but it is not going to be installed`

The Ubuntu APT `npm` package conflicts with NodeSource Node.js v22 (which bundles its own npm). Do **not** install `npm` via APT. Use the one bundled with Node:

```bash
node -v && npm -v  # Both should work already
```

---

## 📚 Resources

| Resource | Link |
|---|---|
| Arc Homepage | [arc.network](https://www.arc.network/) |
| Documentation | [docs.arc.network](https://docs.arc.network/) |
| Block Explorer (Public Testnet) | [testnet.arcscan.app](https://testnet.arcscan.app) |
| Faucet (Testnet USDC) | [faucet.circle.com](https://faucet.circle.com) |
| Bug Bounty | [hackerone.com/circle-bbp](https://hackerone.com/circle-bbp) |
| Circle Developer Console | [console.circle.com](https://console.circle.com/signin) |
| Circle Developer Docs | [developers.circle.com](https://developers.circle.com) |
| arc-node GitHub | [circlefin/arc-node](https://github.com/circlefin/arc-node) |

---

## 📝 Notes

- This guide covers the **local testnet only** — not the public testnet node (which requires 64 GB RAM + 1 TB NVMe SSD)
- Arc is currently in **public testnet phase** — mainnet beta expected in 2026
- Bug bounty testing **must** happen on a local testnet. Do not test against the public testnet.
- Arc node software is **alpha** — expect occasional instability

---

*Maintained independently.
# 🌐 Connect with Me

## 💬 Discord

**Username:** `atkw3`
[Join me on Discord](https://discord.com/channels/@me)

## 🐦 Twitter (X)

[Follow me on Twitter](https://x.com/ATIKURR420)

---

⭐ Feel free to reach out or follow for updates!
.*
