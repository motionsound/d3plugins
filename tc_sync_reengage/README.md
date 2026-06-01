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

1. Pick your **Transport** (the one to monitor + re-engage). The dropdown only lists transports that
   have a **timecode device** patched, and if there's just one it's selected for you — so usually
   there's nothing to pick. *(If no transport has a timecode device yet, all are shown so you're not
   stuck; hit ↻ to refresh after patching one.)*
2. **Frame rate** *(Advanced)* — auto-detected from the transport's timecode device (its SMPTE clock
   type) the moment you pick a transport, so you normally don't touch it. Hit **Detect** to re-read it.
   *(If the transport has no timecode device patched, it falls back to the transport's own SMPTE setting.)*
   - **Override** *(checkbox)* — tick this to stop auto-detecting and set the frame rate manually (for
     the rare case where your gap math needs a different rate than the incoming timecode). Your choice
     sticks across reloads and transport changes. Un-tick it to snap back to auto-detect.
3. **Sync tolerance** *(Advanced)* — there's an inherent ~1–2 frame offset (the two clocks are sampled a
   hair apart), so about **3 frames** is a sensible default. If your project shows desync more often than
   it should, try 4 or 5.

---

## LiveUpdate Command Bus (optional) — drive it from Companion / a stream deck

Lets a stream deck **re-engage** the transport and show a live **sync light + gap**, entirely over
LiveUpdate — no OSC, no bridge. Works locally or from another machine.

### 1) disguise side — one click
In the **LiveUpdate command bus** section, click **Add Device**. That's it — the plugin creates its
own ExpressionVariables device and the variables it needs. Nothing to type, nothing to configure.

- Device created: **`expressionvariablesdevice:plugincmdbus_tcsync`**
- Variables: **`tcsync_cmd`** (triggers re-engage), **`tcsync_status`** (sync state, 1/0),
  **`tcsync_gap`** (live gap in frames)

Click **Verify Device** any time to confirm it's there, then make sure **Enable command bus** is
checked and hit **Apply / reconnect** — the status line should read `bus: connected`.

### 2) Companion side — import the template
1. Add a **Disguise: LiveUpdate** connection — **Host** = the Director's IP, **Port** `80`.
2. Import **`tc_sync_reengage.companionconfig`** (in this plugin's folder) onto an **unused page** in Companion.
3. **Copy the buttons** to wherever you want them.

The imported buttons are already wired to the device's variables — a **Re-Engage** button and a **Sync**
button that shows the live gap and colors itself green/red by sync state. That's the whole setup.

> **Tip:** open Companion's **Variables → liveupdate** section to confirm values are arriving from disguise.

---

## Troubleshooting

- **Command bus won't connect?** Click **Verify Device** — if it isn't found, click **Add Device**. If it
  still won't connect, uncheck **Enable command bus** to stop trying; the sync monitor keeps working on
  its own connection.
- **Companion shows nothing?** Check the **Disguise: LiveUpdate** connection is online (Host = Director
  IP, Port `80`), and that **Add Device / Verify Device** succeeded in the plugin.
- **A button shows the wrong value** → the variable indices in the button paths (`[0]` = `tcsync_cmd`,
  `[1]` = `tcsync_status`, `[2]` = `tcsync_gap`) are specific to a freshly-created `plugincmdbus_tcsync`.
  If you've added/removed/reordered variables on the device, those indices can shift — re-check the
  variable order in Device Manager.
- **Always shows NOT SYNCED** → raise the **Sync tolerance** a frame or two (there's a small inherent offset).

---

## Advanced — what the command bus actually does

You don't need this to use the plugin; it's here for troubleshooting, or if you'd rather build the
Companion buttons by hand instead of importing the template.

The command bus is just an **ExpressionVariables device** in disguise acting as a shared mailbox. The
plugin talks to it over disguise's **LiveUpdate** WebSocket (live push, no polling):

- **`tcsync_cmd`** *(index `[0]`)* — the plugin watches this. When Companion sets it non-zero, the plugin
  fires **Re-Engage**, then resets it to `0` so the next press works the same way.
- **`tcsync_status`** *(index `[1]`)* — the plugin **writes** `1` when synced, `0` when not. Use it to
  color a button.
- **`tcsync_gap`** *(index `[2]`)* — the plugin **writes** the live gap in frames (smoothed so it doesn't
  flicker). Use it to show the number; pair with `tcsync_status` for the colour.

**Add Device** does it all programmatically: creates the device resource, adds it to the project's active
Device List, and adds the three Float variables in that order. Each command-bus plugin makes its own
device with uniquely-named variables, so nothing collides (disguise allows only one variable of a given
name per project) and the object ref / indices are identical on every machine.

### Manual button setup (only if you're not importing the template)

All buttons use **Object Path** `expressionvariablesdevice:plugincmdbus_tcsync`.

**Re-Engage**
- Feedback `LiveUpdate Variable`: Variable Name `tcsync_cmd`, Property Path `object.container.variables[0].defaultFloat`
- Press action `liveupdate: Set to Disguise (Number)`: Variable Name `tcsync_cmd`, Value `1`
- *(The plugin resets it to 0 after firing, so a plain "set 1" works every press.)*

**Sync + gap (number, coloured)**
- Feedback `LiveUpdate Variable`: Variable Name `tcsync_status`, Property Path `object.container.variables[1].defaultFloat`
- Feedback `LiveUpdate Variable`: Variable Name `tcsync_gap`, Property Path `object.container.variables[2].defaultFloat`
- Set the button text to show the `tcsync_gap` variable.
- Two `internal: Variable: Check value` feedbacks on `tcsync_status`: `= 1` → green, `= 0` → red.

Happy syncing. 🎛️
