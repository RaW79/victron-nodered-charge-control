# Victron Node-RED Charge Control

Intelligent LiFePO4 charge management for Victron systems via DVCC and Node-RED.

## What it does

This flow introduces a time-based full charge strategy that standard Victron configurations do not offer: one full charge to the BMS charge voltage limit (CVL, e.g. 55.2 V or 56.0 V as transmitted by the JK-BMS) per configurable interval — weekly, every two weeks, or any other number of calendar weeks — for cell balancing and SOC calibration. Outside that interval the battery is held at a lower conservation voltage (Float).

Full charge detection is CVL-based: the JK-BMS lowers its CVL once the battery is full. The flow detects this drop and records the event in a persistent history file.

## How it works

```
JK-BMS CVL  ──►  Full Charge Logic  ──►  DVCC MaxChargeVoltage  ──►  Victron GX
                  ↑ Heartbeat (1×/h)
                  ↑ Mode inject
```

1. The JK-BMS continuously broadcasts its Charge Voltage Limit (CVL) via D-Bus.
2. When CVL drops by ≥ `DELTA_V` (0.5 V), the logic detects a completed full charge.
3. If the configured interval has passed since the last full charge, the event is recorded and DVCC switches to `FLOAT_VOLTAGE`.
4. If the interval has **not** passed, DVCC stays at `FLOAT_VOLTAGE` immediately (no full charge needed).
5. A heartbeat re-evaluates the DVCC setting every hour without requiring a CVL event.

## Operating modes

| Mode | Behaviour |
|------|-----------|
| `auto` | Full charge every N calendar weeks, conservation charge otherwise |
| `manual` | Forces full charge immediately (DVCC = 0 V → BMS controls voltage) |
| `bulk` | Permanent conservation charge (DVCC = `FLOAT_VOLTAGE`) |

## Parameters

Edit these at the top of the **Volladung Logic** / **Full Charge Logic** function node:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `DELTA_V` | `0.5` | CVL drop (V) that signals a completed full charge |
| `FLOAT_VOLTAGE` | `53.9` | Conservation charge voltage (V) |
| `FULL_CHARGE_INTERVAL_WEEKS` | `2` | Minimum calendar weeks between full charges |

## Requirements

- Victron GX device (Venus OS)
- Node-RED with [node-red-contrib-victron](https://github.com/victronenergy/node-red-contrib-victron)
- JK-BMS connected via D-Bus (battery service `com.victronenergy.battery/512`)
- DVCC enabled in VenusOS

## Installation

1. Download [`flow/ChargeControl_German.json`](flow/ChargeControl_German.json) or [`flow/ChargeControl_English.json`](flow/ChargeControl_English.json).
2. In Node-RED: **Menu → Import → select file**.
3. Adjust the three parameters at the top of the logic node to match your system.
4. Deploy.

The history file is created automatically at:
`/data/home/nodered/.node-red/history/volladung_history.json`

## Screenshots

![Flow overview](flow-overview-de.png)

## Tested with

Victron Multiplus-II GX · JK-BMS PB2A16S20P · node-red-contrib-victron 1.6.x

---

MIT License
