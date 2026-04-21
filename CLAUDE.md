# vesc_can2tcp — Project Context

## What This Project Does
A Raspberry Pi application that bridges VESC Tool TCP connections to a CAN bus interface,
imitating a VESC Express device. VESC Tool connects via TCP (port 65102) and the bridge
translates bidirectionally between VESC packet protocol and CAN bus frames.

## Architecture Overview
- **TCP Server** (port 65102): Accepts VESC Tool connections, speaks VESC packet framing
- **UDP Announcer** (port 65109): Broadcasts presence for VESC Tool auto-discovery
- **CAN Bridge**: Translates between VESC packets and CAN bus frames (500K, 29-bit extended IDs)
- **Language**: Python 3 with asyncio
- **Target**: Raspberry Pi with `can0` interface (SocketCAN)

## VESC Packet Protocol
Framing: `[start_byte] [length] [payload] [crc16] [stop_byte=0x03]`
- Start byte `0x02`: 1-byte length (payload ≤ 255 bytes)
- Start byte `0x03`: 2-byte length (payload ≤ 65535 bytes)
- Start byte `0x04`: 3-byte length
- CRC: CRC16-CCITT over payload only
- First byte of payload is COMM_PACKET_ID (command type)
- Key commands: COMM_FW_VERSION (0), COMM_GET_VALUES (4), COMM_FORWARD_CAN (34)

## UDP Discovery Format
Broadcast every ~1 second on port 65109:
`{hw_name}::{ip_address}::{tcp_port}`
Example: `VESC Express T::192.168.1.50::65102`

## CAN Bus Protocol
- 29-bit extended CAN IDs: `[unused:B28-B16] [command_id:B15-B8] [vesc_id:B7-B0]`
- 500K baud rate
- Simple commands (SET_DUTY=0, SET_CURRENT=1, etc.): 4-byte big-endian payload
- Complex commands: multi-frame using FILL_RX_BUFFER / PROCESS_RX_BUFFER

## Test Environment

### Machine 1: DESKTOP-2RNMAR5 (Windows 11)
- Development machine, has VESC Tool 6.05
- VESC Tool path: `C:\Users\wittr\Documents\GitHub\easyxas\tools\vesc_tool\vesc_tool_free_windows\vesc_tool_6.05.exe`
- Repo: `C:\Users\wittr\Documents\GitHub\vesc_can2tcp`
- Connected to VESC Express via USB (COM7) and TCP

### Machine 2: wittrpi (Raspbian Bookworm)
- Raspberry Pi with CAN interface `can0`
- Repo: `/home/wittr/dev/vesc_can2tcp`
- Target deployment platform
- Connected via Tailscale + local network

### VESC Express (testExpr)
- ESP32-based, firmware 6.05, CAN ID 2
- TCP on port 65102, WiFi SSID: `<see PRIVATE.md>`
- Connected to CAN bus and USB COM7

### VESC 6 MkVI BLDC
- CAN ID 7 ("Unknown: 7" in VESC Tool)
- Motor controller on same CAN bus

### Power Supply
- EA PS2000B — controllable via MCP server `ps2000b`

## MCP Servers Available
- `ps2000b`: EA PS2000B power supply control
- `windows-screenshot`: Screenshot/click automation for VESC Tool GUI
- `chrome-devtools`: Browser automation
- `Central Server Timescale DB`: Database access

## Key Source References
- VESC Express source: https://github.com/vedderb/vesc_express
- VESC firmware (bldc): https://github.com/vedderb/bldc
- VESC Tool source: https://github.com/vedderb/vesc_tool
- Packet framing: `vesc_tool/packet.cpp`
- Command processing: `bldc/comm/commands.c`
- CAN protocol: `bldc/comm/comm_can.c`

## Important Notes
- This is a PUBLIC repo — no passwords, tokens, IPs, or secrets in code
- Use config files with examples/templates for any environment-specific values
- The VESC Express we're imitating uses port 65102 (TCP) and 65109 (UDP broadcast)
- VESC Tool's default TCP port is 65102

## Prior Work (in `old/` and `vesc_github_projects/`)
See `docs/PRIOR_WORK.md` for full analysis. Key reusable components:
- `old/vesc_can2tcp.bck/datatypes.py` — complete CAN_PACKET_ID + COMM_PACKET_ID enums (production-quality)
- `old/vesc_can2tcp.bck/crc.py` — CRC16 lookup table matching VESC firmware
- `old/vesc_can2tcp.bck2/main.py` — VESCCANBridge coordinator pattern
- `vesc_github_projects/vesc_fw_bridge_integration.py` — FW_VERSION response + request detection
- `old/nano_vesc_brigde/` — Arduino C++ version with working UDP broadcast + CAN ping

Critical gap in ALL prior versions: **COMM_FORWARD_CAN** (ID 34) was never implemented.
Also: HW_TYPE was incorrectly set to 12 in old code. Correct values: 0=VESC, 2=CUSTOM_MODULE, 3=VESC_EXPRESS.

## LLM Collaboration
- **Warp (Oz)**: Primary implementation — code, shell commands, testing, PRs
- **Claude**: Architecture review, protocol deep-dives, code review
- **ChatGPT**: Supplementary research (manual via human)
- Shared context via: `docs/PROTOCOL.md`, `docs/DECISIONS.md`, `docs/PRIOR_WORK.md`, commit messages
- See also: [AGENTS.md](AGENTS.md) — coding standards and agent workflow guide
