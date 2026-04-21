# AGENTS.md — Agent Workflow Guide

Cross-reference: [CLAUDE.md](CLAUDE.md) for full project context, protocol specs, and test environment.

## Agents & Roles
- **Warp (Oz)**: Primary implementation — code, shell, testing, PRs
- **Claude**: Architecture review, protocol deep-dives, code review
- **ChatGPT**: Supplementary research (via human relay)

## Coding Standards
- Python 3 + asyncio throughout — no threads, no blocking calls
- No hardcoded secrets, IPs, or SSIDs — use `config.toml` (gitignored); see `config.example.toml`
- Private environment values go in `PRIVATE.md` (gitignored)
- Follow existing module boundaries: packet framing, CAN, TCP, UDP are separate concerns

## Architecture at a Glance
| Component | Port | Notes |
|---|---|---|
| TCP Server | 65102 | VESC packet framing; speaks to VESC Tool |
| UDP Announcer | 65109 | Broadcast `hw_name::ip::port` every ~1s |
| CAN Bridge | can0 | 500K, 29-bit extended IDs |

CAN ID layout: `[unused:B28-B16][command_id:B15-B8][vesc_id:B7-B0]`

## Key Reusable Modules (from `old/`, which is gitignored)
| File | Location | What it provides |
|---|---|---|
| `datatypes.py` | `old/vesc_can2tcp.bck/` | Complete `CAN_PACKET_ID` + `COMM_PACKET_ID` enums |
| `crc.py` | `old/vesc_can2tcp.bck/` | CRC16-CCITT lookup table matching VESC firmware |
| `main.py` | `old/vesc_can2tcp.bck2/` | `VESCCANBridge` coordinator pattern |
| `vesc_fw_bridge_integration.py` | `vesc_github_projects/` | FW_VERSION response + request detection |

Always check these before writing new code — they are production-quality.

## Critical Implementation Gap
**COMM_FORWARD_CAN (ID 34) is missing from all prior versions.** This is the primary command
VESC Tool sends to reach devices on the CAN bus. See `docs/DECISIONS.md` ADR-005 for the
multi-frame protocol details (FILL_RX_BUFFER + PROCESS_RX_BUFFER).

## Common Pitfalls
- `HW_TYPE` must be `3` (VESC_EXPRESS), not `12` as used in old code
- VESC Tool expects the FW_VERSION response to match exactly — see `docs/PROTOCOL.md`
- Multi-frame CAN: large payloads use FILL_RX_BUFFER (0x21) chunks then PROCESS_RX_BUFFER (0x23)
- Selecting "Unknown: 7" in VESC Tool CAN panel can freeze sidebar — restart VESC Tool if stuck

## Architecture Decision Records
See `docs/DECISIONS.md` for ADR-001 through ADR-005 (language choice, identity imitation,
config strategy, packet streaming, COMM_FORWARD_CAN design).

## Test Environment
See [CLAUDE.md](CLAUDE.md#test-environment) for machine details.
Private values (IPs, SSID, UUID) are in `PRIVATE.md` (gitignored).
