# Multi Transport Clocks

A Disguise plugin built by Jared McCammon that shows **Section clocks and Note clocks for every transport
at once**, stacked one per transport. It's the companion to **Video Countdowns** — on a complicated
multi-transport show, hide that plugin's transport-clocks panel and use this one to watch all your
transports' timing side by side.

Clocks are pushed over Disguise's **LiveUpdate**, so they tick in real time.

---

## What you see

One block per transport, each with:

- The transport's **track** and live **Current Time**.
- **SECTION** row: current section name, time **since** the last section break, **UNTIL** the next section
  (name + time), and a **countdown**.
- **NOTE** row: the same, but for notes.

Countdown colour: **16+** green · **6–15** yellow · **5 4 3 2 1** red · **▶** over 99 seconds · **—** when
there's nothing to count to.

---

## Section clocks vs Note clocks

Both show "time since" / "time until" plus a countdown — they use different markers:

- **Section clocks** use your **section breaks**. The current section's name is the note placed *at* the
  break (an unnamed break is auto-numbered "Section N"); the countdown runs to the next section break.
- **Note clocks** use **notes you've added in the timeline** — *including notes that are not at a section
  break*. They track the previous note and count down to the next note, wherever it sits.

So a note **on** a break names that section; a note **mid-section** shows up only in the Note clocks.

---

## Tips

- Transports populate automatically — open the plugin and every transport gets its own block.
- Pair it with **Video Countdowns** (turn that plugin's **clocks** toggle off) for a clean split: countdowns
  for all videos in one window, all-transport timing in this one.

Happy clocking. 🎛️
