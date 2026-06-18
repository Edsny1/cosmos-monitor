# cosmos-monitor

**Multi-chain Cosmos validator TUI dashboard** — by [Edsny1](https://github.com/Edsny1)

```
╔══════════════════════════════════════════════════════════════════╗
║                   OSHVANK  Validator Dashboard                   ║
║         [Push Chain]  [Celestia]  [Lumera]  [Osmosis]  ...      ║
╚══════════════════════════════════════════════════════════════════╝
```

A single terminal dashboard that monitors **all your Cosmos validator nodes simultaneously** — no more staring at raw log streams. Switch between chains with a single key press. Ports are read automatically from each chain's `config.toml` so no manual configuration is needed even when you use custom ports to avoid conflicts.

---

## Features

- **Auto-detects chains** from standard home directories (`~/.pchain`, `~/.celestia-app`, `~/.lumera`, `~/.osmosisd`, `~/.gaiad`, …)
- **Reads ports from `config.toml` / `app.toml`** — works with custom ports (no conflicts)
- **Per-chain colored tabs** — push through instantly between nodes
- **Live data every 5 seconds** via local RPC + REST endpoints
- **Validator table** with paginated list of all network validators
- **Tail logs** directly in the dashboard
- **Process monitoring** — PID, uptime, memory, disk
- **Branded with your ANSI logo** — Edsny1 branding built in

---

## Supported Chains

| Chain | Home Dir | Default Binary |
|---|---|---|
| Push Chain | `~/.pchain` | `pchaind` |
| Celestia | `~/.celestia-app` | `celestia-appd` |
| Lumera | `~/.lumera` | `lumeractl` |
| Osmosis | `~/.osmosisd` | `osmosisd` |
| Cosmos Hub | `~/.gaiad` | `gaiad` |
| Axelar | `~/.axelard` | `axelard` |
| Stride | `~/.stride` | `strided` |
| Nolus | `~/.nolusd` | `nolusd` |
| Any Cosmos chain | `--home <dir>` | auto |

---

## Requirements

- Python **3.11+**
- Linux (Ubuntu 20.04+, Debian 11+) or macOS
- The validator node(s) must be **running locally** (RPC accessible on localhost)

---

## Installation

### One-line install (recommended)

```bash
bash <(curl -sSL https://raw.githubusercontent.com/Edsny1/cosmos-monitor/main/install.sh)
```

### Manual install via pip

```bash
pip install git+https://github.com/Edsny1/cosmos-monitor.git
```

### Install from source (development)

```bash
git clone https://github.com/Edsny1/cosmos-monitor.git
cd cosmos-monitor
bash install.sh --dev
```

---

## Usage

```bash
# Auto-detect all chains on the server
cosmos-monitor

# Monitor a specific chain
cosmos-monitor --home ~/.pchain
cosmos-monitor --home ~/.celestia-app

# Monitor multiple specific chains
cosmos-monitor --home ~/.pchain --home ~/.lumera

# List detected chains without launching TUI
cosmos-monitor --list

# Set custom refresh interval (default: 5 seconds)
cosmos-monitor --refresh 10

# Show version
cosmos-monitor --version
```

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `Tab` / click | Switch between chain tabs |
| `←` / `→` | Page through validator list |
| `r` | Force refresh now |
| `h` | Show help popup |
| `q` / `Ctrl+C` | Quit |

---

## Port Detection

cosmos-monitor reads port configuration **automatically** from each chain's config files:

| Port | Source file | Config key |
|---|---|---|
| RPC | `config/config.toml` | `[rpc] laddr` |
| P2P | `config/config.toml` | `[p2p] laddr` |
| gRPC | `config/app.toml` | `[grpc] address` |
| REST | `config/app.toml` | `[api] address` |

**Example** — if you run Push Chain on ports `54xxx` and Lumera on `17xxx`:

```toml
# ~/.pchain/config/config.toml
[rpc]
laddr = "tcp://0.0.0.0:54657"

[p2p]
laddr = "tcp://0.0.0.0:54656"
```

```toml
# ~/.lumera/config/config.toml
[rpc]
laddr = "tcp://0.0.0.0:17657"
```

cosmos-monitor will pick up `:54657` for Push Chain and `:17657` for Lumera automatically — no extra configuration needed.

---

## Adding a New Chain

If your chain is not in the built-in list, just pass its home directory:

```bash
cosmos-monitor --home ~/.mychain
```

The tool will read `chain_id` from `genesis.json` and ports from `config.toml` / `app.toml` automatically.

To permanently add it to the auto-detection list, open a Pull Request or add it to `cosmos_monitor/chain.py`:

```python
KNOWN_CHAINS[".mychain"] = {
    "name":   "My Chain",
    "denom":  "MYC",
    "binary": "mychaind",
    "color":  "bright_yellow",
}
```

---

## Dashboard Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  HEADER         [Push Chain] [Celestia] [Lumera]  clock        │
├────────────────────┬────────────────────────────────────────────┤
│                    │  ◈ NODE STATUS                             │
│   OSHVANK          │  Process ✓ Running (pid 1234)              │
│   (ANSI logo)      │  RPC     ✓ :54657 listening                │
│                    │  Uptime  4d 12h                            │
│   Validator        │  Memory  22.1%                             │
│   Dashboard        │  ⛓ CHAIN STATUS                            │
│                    │  ████████████████░░░░  100%                │
│                    │  Blocks  17,592,655                        │
├────────────────────┴────────────────────────────────────────────┤
│  🌐 NETWORK STATUS          │  🏛 MY VALIDATOR                  │
│  Node ID  919771f0...       │  OshVanK  [BONDED]                │
│  Moniker  OshVanK           │  Power    106                     │
│  Peers    4 connected       │  Comm.    10%                     │
│  Latency  146ms             │  Rewards  0.44 PC ⚡              │
├─────────────────────────────────────────────────────────────────┤
│  📋 VALIDATORS (page 1/32, total 160)   ← → to page            │
│  NAME              STATUS   STAKE    COMM%  REWARDS  OUTSTANDING│
│  ⊞ OshVanK [Me]   BONDED   106      10%    0.04     0.44       │
│  Validator Valhalla BONDED  1.50B   10%    1.18M    11.88M     │
│  …                                                              │
├─────────────────────────────────────────────────────────────────┤
│  📜 LOGS                                                        │
│  7:48AM INF finalized block hash=921A57... height=17592657      │
│  7:48AM INF executed block  app_hash=5E26DC... height=17592657  │
├─────────────────────────────────────────────────────────────────┤
│  q quit  ←/→ page  r refresh  h help                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting

**Chain not detected**
```bash
cosmos-monitor --list
# If missing, pass the home dir explicitly:
cosmos-monitor --home ~/.mychain
```

**RPC connection refused**
- Make sure the node is running: `systemctl status your-node`
- Check the port in `~/.yourchain/config/config.toml` under `[rpc] laddr`
- Verify the port is bound: `ss -tlnp | grep 26657`

**Logo not showing / blank**
- The ANSI logo requires a terminal with 256-color support
- Test with: `cat assets/logo.ansi`
- iTerm2, Kitty, Alacritty, and most modern terminals work fine
- PuTTY may need color mode set to "xterm-256color"

**Python version too old**
```bash
python3 --version   # need 3.11+
# Ubuntu 20.04:
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.11
```

---

## Project Structure

```
cosmos-monitor/
├── cosmos_monitor/
│   ├── __init__.py      # version
│   ├── cli.py           # entry point & argument parsing
│   ├── chain.py         # auto-detection & config.toml parsing
│   ├── fetcher.py       # RPC/REST data fetching
│   └── dashboard.py     # Textual TUI widgets & layout
├── assets/
│   └── logo.ansi        # Oshvank ANSI logo
├── docs/
│   └── GITHUB_SETUP.md  # GitHub repo setup guide
├── install.sh           # one-line installer
├── pyproject.toml       # Python packaging
└── README.md
```

---

## License

MIT — see [LICENSE](LICENSE)

---

*Built by [Edsny1](https://github.com/Edsny1) — validator on Push Chain, Celestia, Lumera and more.*
