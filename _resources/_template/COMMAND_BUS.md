# LiveUpdate Command Bus — Companion / stream-deck setup (reusable)

Any plugin built from this template that enables the **LiveUpdate Command Bus** can be driven from
Companion / a stream deck — **bridge-free, no OSC**. Companion writes a value into a disguise
**ExpressionVariables device** variable; the plugin reads it over LiveUpdate (push) and runs an action.
The plugin can also **write values back** (e.g. a status) for Companion to read and display.

Ship this file (or paste these steps) with any command-bus plugin.

---

## How it works (one line)

> Companion → *Set to Disguise* (LiveUpdate) a device variable → the plugin (subscribed via LiveUpdate)
> sees the change → runs your action. Read-back works the same way in reverse.

No layers, no OSC, no bridge. Works from another machine — just point Companion's **Host** at the
disguise Director's IP.

---

## 1) disguise: make the command channels

- **Device Manager → add an ExpressionVariables device.** Its name matters (example: **`vardevice`**).
- Add a **Float variable per channel** you need — e.g. one to *trigger* something and one for a
  *status* read-back.
- Note each variable's **index** = its order in the device: first = **0**, second = **1**, …
  (Companion addresses variables by index.)

## 2) Plugin: point it at the device

- **Device (object ref):** `expressionvariablesdevice:<device name>`  (e.g. `expressionvariablesdevice:vardevice`)
- Enter the **variable name(s)** the plugin asks for, enable the command bus, and apply/reconnect.

## 3) Companion: add a *Disguise: LiveUpdate* connection

Set **Host** to the disguise Director's IP, **Port** `80`.

### To WRITE a value (trigger the plugin)
On a button:
1. **Feedbacks → `LiveUpdate Variable`:**
   - **Variable Name:** a Companion name (e.g. `cmdvar`)
   - **Object Path:** `expressionvariablesdevice:<device name>`
   - **Property Path:** `object.container.variables[<index>].defaultFloat`  *(index = the variable's 0-based position in the device)*
2. **Press actions → `liveupdate: Set to Disguise (Number)`:**
   - **Variable Name:** `cmdvar`  *(the Companion variable from step 1)*
   - **Value:** your command number (often `1`).

> Many plugins make a trigger **momentary** — they fire on a change to a non-zero value and then reset
> it to `0` — so a plain "set 1" button works on every press.

### To READ a value (display plugin status on a button)
1. **Feedbacks → `LiveUpdate Variable`** with the Object/Property pointing at the status variable
   → creates `$(liveupdate:<name>)`.
2. **Feedbacks → `internal: Variable: Check value`** (one or more) to color the button by value
   (e.g. `= 1` → green, `= 0` → red).

> Confirm values are flowing in **Companion → Variables → liveupdate**.

---

## Gotchas

- **Index must match.** `object.container.variables[N].defaultFloat` — `N` is the variable's 0-based
  position in the device (first = `[0]`).
- **Name not found** → the variable name doesn't match one on that device.
- **Float values.** `defaultFloat` is the live, settable value for a Float variable; send numbers.
- **One device, many channels.** Add more variables for more commands/status read-backs.
- **Read-only?** Computed/keyframed properties reject `set`; plain device-variable floats are writable.

---

## Plugin author notes (template)

The template's commented **OPTIONAL LIVEUPDATE COMMAND BUS** block already implements the read side:
it subscribes to your named channels and calls `onCommand(name, value)` on change. To enable, set
`LU_DEVICE` / `LU_CHANNELS`, fill in `onCommand`, and uncomment the block + `startCommandBus();`.
Read values by name with
`next((v.defaultFloat for v in object.container.variables if v.name=="cmdN"), None)`; **write** by index
with `{set}` on `object.container.variables[N].defaultFloat`.
