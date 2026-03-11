# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dionaea is a low-interaction honeypot designed to catch malware. It is the successor to nepenthes, embedding Python as a scripting language, using libemu for shellcode detection, and supporting IPv6 and TLS.

## Build Commands

Dionaea uses CMake and requires out-of-source builds:

```bash
# Create build directory and build
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/opt/dionaea ..
make
make install
```

## Code Quality

Run the style checker:
```bash
tox -e style
```

Or use pre-commit directly:
```bash
pre-commit run --all-files
```

Install pre-commit hooks (recommended):
```bash
pre-commit install
```

## Architecture

### Core (C)

The core is written in C and provides:
- Event-driven architecture using libev (`src/`)
- Connection handling for TCP, UDP, TLS, DTLS (`src/connection*.c`)
- Module loading system (`src/modules.c`)
- Incident reporting system (`src/incident.c`)
- DNS resolution, logging, signal handling

Key header files are in `include/`. The main entry point is `src/dionaea.c`.

### Python Modules (`modules/python/dionaea/`)

Services and incident handlers are implemented in Python:

- **Services**: Extend `ServiceLoader` class in `__init__.py`
  - Implement `start(cls, addr, iface=None, config={})` class method
  - Examples: `smb/smb.py`, `http.py`, `ftp.py`, `mysql/mysql.py`

- **Incident Handlers**: Extend `IHandlerLoader` class in `__init__.py`
  - Implement `start(cls, config={})` class method
  - Examples: `hpfeeds.py`, `log_json.py`, `fail2ban.py`

### C Modules (`modules/`)

Optional C extension modules:
- `emu/` - Shellcode emulation using libemu
- `curl/` - HTTP client functionality
- `nfq/` - Netfilter queue integration
- `pcap/` - Packet capture
- `nl/` - Netlink interface monitoring

### Configuration

Configuration files are YAML-based:
- Main config: `dionaea.cfg` (generated from `conf/dionaea.cfg.cmake`)
- Service configs: `conf/services/*.yaml`
- IHandler configs: `conf/ihandlers/*.yaml`

At runtime, configs are loaded from patterns specified in the `module` section.

### Incident System

The incident system provides event-driven communication between components:
- Incidents have an origin string (e.g., `"dionaea.module.nl.addr.new"`)
- Handler patterns use glob matching via `GPatternSpec`
- Data types: bytes, string, int, ptr, list, dict

## Testing

- Functional tests are in `tests/` (SIP, MySQL, printer protocols)
- CI builds via Drone CI (`.drone.yml`)
- Docker images built via GitHub Actions

## Docker

Official images: `dinotools/dionaea`

Key directories in container:
- Config: `/opt/dionaea/etc/dionaea`
- Data: `/opt/dionaea/var/lib/dionaea`
- Logs: `/opt/dionaea/var/log/dionaea`

## Code Style

- Python: Follow PEP 8, max line length 120 characters
- C: Follow existing style in the codebase
- All files must have SPDX license headers
