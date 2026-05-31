# Timecode Sync + Re-Engage

A Disguise plugin built by Jared McCammon that monitors a transport's **incoming timecode** against its
**timecode position** so you can see, at a glance, whether the playhead is **synced** to
incoming timecode — with several ways to re-engage when it drifts.

Everything that can be is done over disguise's **LiveUpdate** WebSocket (live push, no polling).

---

## What you see

- **SYNCED / NOT SYNCED** light (pinned at the top) plus the current **gap** in frames.
- **Clocks:** *Incoming TC* and *TC position* side by side.
- Collapsible sections — collapse them for a tiny during-show window. It remembers what you
  collapsed, so it stays compact between sessions.

> *Incoming timecode* is the transport-wide TC coming in. *Timecode position* is the playhead's
> position relative to the timecode tags on the track (not absolute track time). When they match,
> you're synced.

---

## Re-engaging — three ways

1. **Re-Engage now** — click the button; it disengages then immediately re-engages the chosen transport.
2. **Auto Re-Engage on de-sync** *(checkbox)* — set your drift threshold (Advanced → *Sync tolerance*,
   in frames). If the gap exceeds it, the plugin re-engages automatically and keeps watching — it
   re-tries while out of sync (~5 s apart) and re-arms once you're synced again.
3. **Command Bus** — trigger Re-Engage (and a sync status light) from a stream deck / Companion over
   LiveUpdate. See setup below.

---

## Setup

1. Pick your **Transport** (the one to monitor + re-engage).
2. In **Advanced**, set **Frame rate** and **Sync tolerance** (frames).
   - There's an inherent ~1–2 frame offset (the two clocks are sampled a hair apart), so a tolerance
     of about **3 frames** is a sensible default. 
     If your project is showing desync more often than you think it should, try increasing this to 4 or 5.

---

## LiveUpdate Command Bus (optional) — drive it from Companion / a stream deck

Lets a stream deck **re-engage** the transport and show a **red/green sync light**, entirely over
LiveUpdate — no OSC, no bridge. Works locally or from another machine.

### 1) disguise side
- **Device Manager → add an ExpressionVariables device.** Its name matters — this example uses **`vardevice`**.
- Add **two Float variables**: one to trigger re-engage, one for sync status. To match the plugin's
  hints, name them **`reengagecmd`** and **`syncstatus`**.
- Note the **order** you create them in — the first variable is index **0**, the second is **1**, etc.
  (you'll need this for Companion).

### 2) Plugin side — *LiveUpdate command bus* section
- **ExpressionVariables device:** `expressionvariablesdevice:vardevice`  *(i.e. `expressionvariablesdevice:<your device name>`)*
- **Re-engage variable:** `reengagecmd`
- **Sync variable:** `syncstatus`
- Make sure **Enable command bus** is checked, then **Apply / reconnect**. The status line should read
  `bus: connected`.

### 3) Companion side
Add a **Disguise: LiveUpdate** connection and set its **Host** to the disguise Director's IP (Port `80`).

**Re-Engage button**
- Add a button; label it whatever you like (e.g. "Re-Engage").
- **Feedbacks tab → add `LiveUpdate Variable`:**
  - **Variable Name:** `reengagecmd`  *(this names the Companion variable; matching the disguise name keeps it clear)*
  - **Object Path:** `expressionvariablesdevice:vardevice`
  - **Property Path:** `object.container.variables[0].defaultFloat`
    *(the `[0]` is the variable's index in the device — first variable = 0; if it were the 7th, use `[6]`)*
- **Press actions → add `liveupdate: Set to Disguise (Number)`:**
  - **Variable Name:** `reengagecmd`  *(the Companion variable from the feedback above)*
  - **Value:** `1`
- Press it → the transport re-engages. *(The plugin fires on the change and resets the value to 0 for
  you, so a plain "set 1" button works every press.)*

**Sync status button (auto red/green)**
- Add a button; label it e.g. "Sync".
- **Feedbacks tab → add `LiveUpdate Variable`:**
  - **Variable Name:** `syncstatus`
  - **Object Path:** `expressionvariablesdevice:vardevice`
  - **Property Path:** `object.container.variables[1].defaultFloat`
    *(second variable = index 1; adjust as above)*
- **Add two more feedbacks → `internal: Variable: Check value`:**
  - Find your `syncstatus` variable (search for it).
  - If it **= 1** → background **green** (synced). If it **= 0** → background **red** (not synced).
- The button now colors itself live from disguise.

> **Tip:** open Companion's **Variables → liveupdate** section to confirm the values are arriving from disguise.

---

## Troubleshooting

- **Command bus won't connect?** Uncheck **Enable command bus** to stop trying — the sync monitor keeps
  working on its own connection.
- **"var not found" in the log** → the variable name doesn't match one on the device (check spelling and
  that it lives on *that* device).
- **Wrong thing fires / nothing happens** → the **Property Path index** must match the variable's position
  in the device (0-based: first = `[0]`, second = `[1]`, …).
- **Always shows NOT SYNCED** → raise the **Sync tolerance** a frame or two (there's a small inherent offset).

Happy syncing. 🎛️
