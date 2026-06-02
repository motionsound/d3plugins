# Video Countdowns

A Disguise plugin built by Jared McCammon that shows a compact, colour-coded **countdown for every
video layer currently playing** — across all of your transports — plus Section and Note clocks for a
transport you pick.

Everything live is pushed over Disguise's **LiveUpdate** (no polling for the clocks), so the
countdowns tick in real time.

---

## What you see

- **One row per playing video**, each showing two clocks and a big **countdown** number on the right.
  - Countdown colour: **16+** green · **6–15** yellow · **5 4 3 2 1** red · **▶** when more than 99 seconds
    remain · **—** when muted or finished.
- A coloured **tag** on each row showing which transport the video is on (one colour per transport).
- **Active / Inactive** state: `MUTED`, `INACTIVE — BRIGHTNESS 0`, or `INACTIVE — TRANSPORT BRIGHTNESS 0`
  (muted clips drop to the bottom; the others stay put and keep counting).
- A **Transport clocks** panel up top (toggleable) with Section and Note clocks for the picked transport.

---

## Clip vs Layer  *(the timing mode — top-right dropdown)*

This sets what every countdown and clock measures:

- **Layer** — uses the layer's **position on the timeline**. "Layer time" is how long the layer has been
  under the playhead; the countdown is the time until the layer's block on the timeline ends.
- **Clip** — uses the **actual frames of the playing video file**. If the clip is **looping**, has a
  **changed speed**, ping-pongs, etc., this will be a **different value than Layer** (e.g. a looping clip
  that's on its third loop shows where it is *within the file*, not how long the layer has been up).

Pick whichever matches how you think about your cues. **Layer** is the common default; **Clip** is for when
you care about the file's own playhead.

---

## Section clocks vs Note clocks  *(the top panel)*

Both show a "time since" / "time until" pair plus a countdown — they just use different markers:

- **Section clocks** use your **section breaks**. The current section's name is the note placed *at* the
  break (an unnamed break is auto-numbered "Section N"); the countdown runs to the next section break.
- **Note clocks** use **notes you've added in the timeline** — *including notes that are not at a section
  break*. They track the previous note and count down to the next note, wherever it sits.

So a note sitting **on** a break names that section (Section clocks); a note sitting **mid-section** is
tracked only by the Note clocks.

---

## Toggles  *(top-right of the Videos list)*

- **clocks** — show/hide the whole top Transport-clocks panel (turn it off and use the companion
  *Multi Transport Clocks* plugin to watch every transport's timing at once).
- **sort ↕** — keep the soonest countdown at the top (re-orders live).
- **Hide \<transport\>** *(bottom, only with multiple transports)* — hide all of a transport's videos.

---

## Good to know

- **Layer / Section / Note clocks work on every transport.** The **Clip** mode and mute detection rely on
  render data that Disguise only provides for one transport at a time, so on other transports the Clip
  clocks show **—** (use **Layer** there). Switch the top picker to change which transport's section/note
  clocks you're watching — the video list always shows them all.

Happy countdowning. 🎬
