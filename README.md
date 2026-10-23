# AedesGuard: Virtual Prototype

**Sime Darby Property University Hackathon 2026**

AedesGuard is a smart storm drain for Sime Darby Property townships. Ordinary roadside drains often hold a shallow pool of water after rain, and Aedes mosquitoes (which spread dengue) breed in it. A field study of 52 storm drains (Souza et al., 2017) found standing water in 92% of them.

AedesGuard fixes this in three layers:

1. **Design:** a sloped V-shaped floor so rainwater drains out instead of pooling.
2. **Sensing:** a wetness probe and an ultrasonic water-level sensor, read by an ESP32 controller.
3. **Action:** if the floor stays wet for **48 hours** (long before larvae can become adults, which takes about 8 days), a pump flushes the drain with harvested rainwater. If it is still wet after **2 flushes**, the maintenance team gets a **"Needs clearing"** alert. Data travels over LoRa radio → gateway → Firebase → dashboard and Telegram.

This repository contains **two interactive prototypes** that let you see all of this working without any hardware.

| | Prototype | What it shows | File |
|---|---|---|---|
| 1 | **Drain Digital Twin** | One AedesGuard drain in 3D, side by side with a normal drain, under the same rain | [`digital-twin.html`](digital-twin.html) |
| 2 | **Township Dashboard** | What the maintenance team sees for a whole township: map, drain status, alerts, event log | [`dashboard.html`](dashboard.html) |

> **Suggested order:** open the Digital Twin first and press **Play guided demo** (about 3 minutes). Then open the Dashboard and press **Run demo storm**.

---

## How to open the prototypes

You need a recent **Google Chrome or Microsoft Edge** on a laptop or desktop, and an **internet connection** (the pages load the 3D engine, map tiles and fonts online). Nothing needs to be installed.

### Option A: open online (easiest)

If the team has turned on GitHub Pages, click:

- **Landing page:** `https://<github-username>.github.io/aedesguard-virtual-prototype/`
- **Digital Twin:** `https://<github-username>.github.io/aedesguard-virtual-prototype/digital-twin.html`
- **Dashboard:** `https://<github-username>.github.io/aedesguard-virtual-prototype/dashboard.html`

### Option B: download and double-click

1. On this repository page, click the green **Code** button → **Download ZIP**.
2. Unzip the folder.
3. Double-click **`digital-twin.html`** or **`dashboard.html`**. It opens in your browser.

> Do not open the files using GitHub's file viewer (it only shows the code). Use Option A or B.

---

## Prototype 1: Drain Digital Twin

A 3D, physics-based model of the drain. The **left** drain is a normal storm drain; the **right** drain is AedesGuard. Both get exactly the same rain.

### The fastest way: guided demo

Press the yellow **Play guided demo** button (top right). A caption box at the bottom of the 3D view walks through 10 steps and moves the camera for you:

| Step | What happens | What to look at |
|---|---|---|
| 1 | Introduces both drains | Left = normal drain, right = AedesGuard |
| 2 | A one-hour storm hits both | Rain falls; controller state turns **RAINING** (flushing is paused during rain) |
| 3 | Rain stops | Normal drain keeps a pool of water; AedesGuard empties within minutes |
| 4 | Leaves wash in and trap a small puddle | Controller turns **WET**; the 48 h clock starts |
| 5–6 | Time skips to hour 48 | Pump turns on, spray jets sweep the floor (**FLUSHING**) |
| 7 | Re-check | Floor is dry → **DRY**; a radio packet animates to Firebase |
| 8 | 9 days with no rain | **Aedes clock** panel: normal drain goes egg → larva → pupa → adult mosquitoes fly out; AedesGuard stays clear |
| 9 | Outlet blocked + another storm | Water backs up → **FLOOD_RISK** alert pops up as a Telegram message |
| 10 | Water cannot leave | **BLOCKAGE** ticket after 6 h; maintenance clears it → back to **DRY** |

Press **Stop** in the caption box at any time to take over.

### Screen layout

- **3D view (left):** drag to rotate, scroll to zoom. Camera buttons below it jump to *Overview*, *AedesGuard*, *Controller*, *Tank + pump* and *Normal drain*. **Labels** toggles part names; **Front wall** toggles a see-through front wall.
- **Top bar:** simulated clock, **Pause/Play**, speed (*1 s = 1 min / 10 min / 1 h*), **+1 day**, **Skip to next event**.
- **Make something happen:** buttons to trigger events yourself (see below).
- **Controller state:** the current state highlighted, with a plain-English description, hours wet (flush at 48 h) and flush count (max 2).
- **What the ESP32 reads:** live sensor values: wetness probe, water level, tank level, pump, supply voltage.
- **Aedes clock (bottom left):** mosquito life-cycle bar for each drain. The black marker shows how far any larvae have grown; the dashed blue line is the 48 h flush.
- **LoRa → gateway → Firebase (bottom middle):** the exact radio payload and database message the real hardware would send. A dot animates along the path on each report (red for alerts).
- **Event log (bottom right):** everything that happened, newest first.

### Try it yourself

| Press | Expected result |
|---|---|
| **Light rain · 1 h** or **Heavy storm · 1 h** | State goes to RAINING, then DRY once the floor drains. The normal drain keeps water. |
| **Leaves wash in** | A puddle is trapped → WET → (press **Skip to next event**) → FLUSHING at 48 h → DRY |
| **SW2 test button** | Forces an immediate flush cycle, like the test button on the real controller |
| **Block outlet strainer**, then **Heavy storm** | FLOOD_RISK during rain, then BLOCKAGE; a Telegram-style alert appears |
| **Maintenance: clear drain** | Clears leaves and blockage; the drain returns to DRY |
| **Empty the tank**, then **Leaves wash in** + **Skip to next event** | At 48 h the controller reports TANK_LOW instead of flushing |
| **Reset both drains** | Starts over |

---

## Prototype 2: Township Dashboard

The control screen for the township maintenance team. It opens with **5 simulated drains** (marked **SIM**) on a map, each with a different built-in fault, and runs the same control logic as the firmware with time sped up (by default 1 simulated hour = 0.25 s, so 48 h ≈ 12 s).

| Drain | Built-in fault | What you will see after a storm |
|---|---|---|
| Drain B, Drain D | Drains normally | Dries on its own; no flush needed |
| Drain A, Drain E | Leaves a puddle | Wet for 48 h → 1 flush → dry |
| Drain C | Strainer clogged | Flood risk during the storm, then 2 failed flushes → **Needs clearing** alert |

### Quick demo (about 1 minute)

1. Press **Run demo storm (3 h rain)** in the *Demo controls* card.
2. Watch the **map pins** and **Drains** table change colour. Drain C turns red (**Flood risk**) while it rains.
3. After the rain, puddle drains count up **Hours wet**, flush at 48 h, and return to **Dry**.
4. Drain C fails both flushes and moves to **Needs clearing**; the **Need clearing** summary box turns red.
5. Click **Drain C** in the table, then press **Maintenance: clear drain**. It returns to Dry, and the event log records *"Cleared"*.

### Screen layout

- **Summary boxes:** % of drains dry, drains that need clearing, drains at flood risk now, flushes run.
- **Map:** one pin per drain, coloured by status. Click a pin to select it. (Simulated drain positions are illustrative.)
- **Drains table:** sorted so the most urgent drain is at the top. Click a row to open its details.
- **Drain details:** water-level chart (yellow band = floor wet, blue band = flushing, red dashed line = flood level), sensor fields, and per-drain controls for simulated drains: change the fault, *Rain here*, *Empty the tank*, *Maintenance: clear drain*.
- **Event log:** every status change in plain English. Alerts have an **Acknowledge** button.
- **Export (CSV):** download the event log or a drain's readings.

### Status colours

| Status | Meaning |
|---|---|
| 🟢 Dry | Floor is dry, nothing to do |
| 🔵 Raining | Inflow detected; flushing is paused |
| 🟡 Wet | Floor still wet after rain; 48 h timer running |
| 🔵 Flushing / Re-checking | Pump running, or checking whether the flush worked |
| 🟠 Tank low | Wet for 48 h but not enough rainwater to flush |
| 🔴 Flood risk | Water rising fast during rain; outlet may be clogged |
| 🔴 Needs clearing | Still wet after 2 flushes; send a crew |
| ⚪ Offline | No data from a real drain for over 2 minutes |

### Settings (optional, not needed for judging)

The yellow banner *"Not connected to Firebase"* is expected. The simulation works without it. **Settings** lets the team connect the dashboard to the live Firebase database (to show the real hardware drain) and to a Telegram group for alerts. These settings are stored only in your own browser.

---

## How the two prototypes connect

```
 AedesGuard drain           Gateway             Cloud              People
 ESP32 + sensors  --LoRa-->  ESP32   --Wi-Fi-->  Firebase  ----->  Dashboard (maintenance team)
 (Digital Twin)              923 MHz             /nodes    ----->  Telegram alerts
```

The Digital Twin simulates **one drain node** in detail. The Dashboard shows **many drains** across a township. When the Digital Twin file is opened directly on a laptop, it also sends its data to the team's Firebase database, so it appears on the Dashboard as a real (non-SIM) drain once the Dashboard is connected to the same database.

## Troubleshooting

| Problem | Fix |
|---|---|
| The page shows code instead of the prototype | You opened it in GitHub's viewer. Use Option A or B above. |
| 3D view or map is blank | Check your internet connection and refresh. Use Chrome or Edge. |
| Everything moves too fast | Twin: pick **1 s = 1 min** in the top bar. Dashboard: change **Speed** to *1 h = 1 s*. |
| I want to start again | Twin: **Reset both drains**. Dashboard: **Reset simulation**. |

## Files

| File | Description |
|---|---|
| `index.html` | Landing page with links to both prototypes |
| `digital-twin.html` | Drain Digital Twin (3D, single drain, guided demo) |
| `dashboard.html` | Township Dashboard (map, status, alerts) |

Reference: Souza, R. L. et al. (2017). *Effect of an intervention in storm drains to prevent Aedes aegypti reproduction in Salvador, Brazil.* Parasites & Vectors, 10, 328.
