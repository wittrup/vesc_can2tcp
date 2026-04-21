# vesc_can2tcp

A Raspberry Pi bridge that exposes VESC CAN bus devices over TCP, imitating a VESC Express.
Allows VESC Tool to connect via TCP/WiFi and communicate with VESCs on the CAN bus.

## Features

- **TCP Server** on port 65102 — drop-in replacement for VESC Express TCP connection
- **UDP Discovery** on port 65109 — VESC Tool auto-discovers the bridge on the network
- **CAN Bus Bridge** — bidirectional translation between VESC packet protocol and CAN frames
- **CAN Forwarding** — supports VESC Tool's CAN Forward feature to address individual VESCs

## Requirements

### Hardware
- Raspberry Pi (any model with network access)
- CAN bus interface (e.g., MCP2515 HAT, Waveshare CAN HAT, USB-CAN adapter)
- CAN bus connection to VESC device(s)

### Software
- Python 3.9+
- `python-can` library
- SocketCAN (`can-utils`)

## Quick Start

### 1. Set up CAN interface
```bash
sudo ip link set can0 up type can bitrate 500000
```

### 2. Install dependencies
```bash
pip3 install python-can
```

### 3. Configure
```bash
cp config.example.toml config.toml
# Edit config.toml with your settings
```

### 4. Run
```bash
python3 -m vesc_can2tcp
```

### 5. Connect from VESC Tool
In VESC Tool: Connection → TCP → enter the Pi's IP address, port 65102 → Connect.
Or use auto-discovery — the bridge broadcasts its presence on the local network.

## Configuration

Copy `config.example.toml` to `config.toml` and adjust:

```toml
[tcp]
port = 65102
bind = "0.0.0.0"

[udp]
discovery_port = 65109
broadcast_interval = 1.0
hw_name = "vesc_can2tcp"

[can]
interface = "socketcan"
channel = "can0"
bitrate = 500000
```

## Project Structure

```
vesc_can2tcp/
├── CLAUDE.md              # LLM project context
├── README.md
├── config.example.toml    # Configuration template
├── docs/
│   ├── PROTOCOL.md        # VESC protocol reference
│   ├── DECISIONS.md       # Architecture decision records
│   └── TEST_SETUP.md      # Test environment documentation
└── src/
    └── vesc_can2tcp/
        ├── __init__.py
        ├── __main__.py    # Entry point
        ├── packet.py      # VESC packet framing/deframing
        ├── can_bridge.py  # CAN bus interface
        ├── tcp_server.py  # TCP server
        ├── udp_discovery.py # UDP broadcast announcer
        └── config.py      # Configuration loader
```

## How It Works

```
VESC Tool ──TCP:65102──▶ vesc_can2tcp ──CAN bus──▶ VESC 6 / VESC 75 / etc.
                              │
                              ├── Parses VESC packet framing
                              ├── Handles COMM_FW_VERSION (identifies as bridge)
                              ├── Translates COMM_FORWARD_CAN to CAN frames
                              └── Forwards CAN status messages back to TCP
```

## License

TBD

## Acknowledgments

- [VESC Project](https://vesc-project.com/) by Benjamin Vedder
- [VESC Express](https://github.com/vedderb/vesc_express) — the device this project imitates
- [VESC Tool](https://github.com/vedderb/vesc_tool) — the client this project is compatible with
