# Victron Node-RED — Charge Control for LiFePO4

> Intelligent charge interval management for Victron systems via DVCC and Node-RED.

![Flow Overview](charge-control/flow-overview-en.png)

---

## Why this flow exists

There are two widely held views on LiFePO4 charging: regular full charges support cell balancing, while frequent exposure to high charge voltages may accelerate degradation. Both views have merit — and most systems force you to pick one fixed voltage, leaving no room for a middle ground.

In summer, high PV yield can push the battery to full charge voltage (RCV) once or even multiple times a day depending on consumption and state of charge. The BMS or Victron reduces to float (RFV) automatically afterwards — but each new charge cycle repeats the process.

This flow introduces a time-based strategy that standard systems do not offer: one full charge to e.g. 56.0 V / 55.2 V (according to CVL from BMS) per calendar week interval *(configurable)* for balancing, and float voltage e.g. 53.9 V *(configurable — overrules BMS floating voltage)* for the remaining days. How beneficial this is depends on your system and preferences — the flow simply makes it possible.

The charge strategy:

| When | Behavior |
|------|----------|
| **Once per configured interval** (default: every 2 calendar weeks) | Charge to full voltage (BMS CVL, e.g. 56.0 V) — full charge for balancing |
| **All other days** | DVCC limits charge to **`FLOAT_VOLTAGE`** (e.g. 53.9 V) — overrules BMS, conservation mode |

Full charges are **saved persistently** to a JSON file so a system reboot never triggers a duplicate full charge.

---

## Operating Modes

| Mode | Behavior |
|------|----------|
| `auto` | Automatic — max. 1 full charge per N calendar weeks (configurable via `FULL_CHARGE_INTERVAL_WEEKS`) |
| `manual` | Force an immediate full charge right now |
| `bulk` | Stay in conservation mode permanently (55.2 V), no full charge |

Switch modes using the **Inject nodes** inside the flow.

---

## How it detects a full charge

The flow reads the **CVL (Charge Voltage Limit)** from the BMS. When the CVL drops by ≥ `DELTA_V` (0.5 V), a full charge is considered complete and logged to the history file.

---

## Parameters

| Parameter | Default | Meaning |
|-----------|---------|---------|
| `DELTA_V` | `0.5` | CVL drop (V) that signals a completed full charge |
| `FLOAT_VOLTAGE` | `55.2` | Conservation charge voltage in V (3.45 V/cell) |
| `FULL_CHARGE_INTERVAL_WEEKS` | `2` | Minimum calendar weeks between full charges |

---

## System Paths

| Function | Victron D-Bus Path |
|----------|--------------------|
| DVCC max charge voltage | `com.victronenergy.settings` → `/Settings/SystemSetup/MaxChargeVoltage` |
| CVL from BMS | `com.victronenergy.battery/512` → `/Info/MaxChargeVoltage` |
| History file | `/data/home/nodered/.node-red/history/volladung_history.json` |

---

## Requirements

- Victron system with **DVCC** support
- **JK-BMS** or compatible BMS that exposes CVL via D-Bus
- **Node-RED** with [`node-red-contrib-victron`](https://flows.nodered.org/node/node-red-contrib-victron) installed
- Write access to the Node-RED data directory

---

## Installation

1. Download the flow:
   - **[`ChargeControl_English.json`](charge-control/flow/ChargeControl_English.json)** (English)
   - **[`ChargeControl_German.json`](charge-control/flow/ChargeControl_German.json)** (Deutsch)
2. Open **Node-RED** in your browser
3. Click the hamburger menu → **Import**
4. Select the downloaded `.json` file and confirm
5. Click **Deploy**
6. Activate the desired mode via the Inject node (`auto` is recommended to start)

---

## Flow Structure

The flow is organized into four groups:

| Group | Purpose |
|-------|---------|
| **INPUTS** | Reads CVL from BMS, mode selector, timer triggers |
| **LOGIK** | Evaluates weekly full-charge logic and history |
| **FILTER** | Prevents redundant DVCC writes |
| **OUTPUT** | Writes DVCC max charge voltage to Victron system |

---

## License

MIT — free to use, modify and share.

---

---

![Flow Übersicht](charge-control/flow-overview-de.png)

## Warum dieser Flow

Zum Thema LiFePO4-Laden gibt es zwei verbreitete Sichtweisen: Regelmäßige Volladungen fördern das Zellbalancing, während häufige hohe Ladespannungen die Alterung beschleunigen können. Beide Ansichten haben ihre Berechtigung — und die meisten Systeme lassen nur eine feste Spannungsvorgabe zu, ohne Spielraum für eine differenzierte Strategie.

Im Sommer kann hohe PV-Leistung die Batterie je nach Verbrauch und Ladezustand ein- oder auch mehrmals täglich auf die Volladespannung (RCV) bringen. Das BMS oder Victron reduziert danach automatisch auf die Floatspannung (RFV) — aber jeder neue Ladezyklus beginnt den Prozess erneut.

Dieser Flow ermöglicht eine zeitbasierte Strategie, die Standardsysteme so nicht bieten: einmal pro konfiguriertem Intervall (Standard: alle 2 Kalenderwochen) eine Volladung auf z. B. 56,0 V / 55,2 V *(gemäß CVL des BMS)* für das Balancing, den Rest der Zeit mit z. B. 53,9 V Floatspannung *(konfigurierbar — überschreibt die BMS-Floatspannung)*. Wie sinnvoll das im eigenen System ist, bleibt der persönlichen Einschätzung überlassen — der Flow macht es schlicht möglich.

Die Ladestrategie:

| Wann | Verhalten |
|------|-----------|
| **1x pro konfiguriertem Intervall** (Standard: alle 2 Kalenderwochen) | Laden auf BMS-Volladespannung (z. B. 56,0 V) — Volladung für Balancing |
| **Alle anderen Tage** | DVCC begrenzt Ladung auf **`FLOAT_VOLTAGE`** (z. B. 53,9 V) — überschreibt BMS, Schonmodus |

Volladungen werden **persistent gespeichert**, damit ein Neustart keine doppelte Volladung auslöst.

## Installation

1. Flow herunterladen:
   - **[`ChargeControl_English.json`](charge-control/flow/ChargeControl_English.json)** (English)
   - **[`ChargeControl_German.json`](charge-control/flow/ChargeControl_German.json)** (Deutsch)
2. **Node-RED** im Browser öffnen
3. Hamburger-Menü → **Import** → Datei auswählen
4. **Deploy** klicken
5. Betriebsmodus über den Inject-Node aktivieren (`auto` zum Starten empfohlen)
