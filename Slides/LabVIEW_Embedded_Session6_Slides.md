---
marp: true
theme: default
paginate: true
size: 16:9
header: '**LabVIEW Embedded** · Session 6'
footer: 'ISEN4 M1 — Bio Med / Smart Energy & Embedded / Robotic'
---

<style>
header {
  color: #2f2f2f;
  font-weight: bold;
}
footer {
  color: #457B9D;
}
img {
  display: block;
  margin: 0 auto;
}
</style>

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 6️⃣ Session 6
## The final boss

### A standalone app, running on its own, no PC attached

---

## Today's plan *(relative to your session start)*

| Elapsed | Block |
|---|---|
| 0:00–0:10 | Welcome — today's goal |
| 0:10–0:40 | Concept 1 — What a build spec actually gives you |
| 0:40–1:50 | Work 1 — Build the `.rtexe`, set it as startup |
| 1:50–2:05 | Break |
| 2:05–2:35 | Work 2 — Power-cycle it, and watch |
| 2:35–3:15 | Work 3 — The flight recorder: log it, then go find it |
| 3:15–3:45 | Wrap-up — docs, tag, ship it |
| 3:45–4:00 | Closing |

---

## 🎯 Today's goal

Last session, the app ran on the MyRIO — but only while LabVIEW stayed connected to it.

Today, it becomes a **real embedded product**: plug in power, and it's already running. No laptop, no LabVIEW, no cable.

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🧩 Concept 1
## What a build spec actually gives you

---

## Interactive vs. standalone

- **Interactive** *(what you did last session)*: LabVIEW on your PC talks to the target continuously, running your VI live
- **Standalone**: your VI hierarchy gets **compiled into an executable** that lives entirely on the target — no host connection needed, ever, once it's running

A **Build Specification** is what turns "code LabVIEW runs for you" into "a program the MyRIO owns."

---

## This is the actual point of "embedded"

Everything since Session 1 — the contract, the modules, the queues — was building toward code that's genuinely **independent**.

Today it stops being a LabVIEW project you run, and becomes a device that does its job.

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 1
## Build the `.rtexe`, set it as startup

---

## 🛠️ Steps

1. In `LVEmbeddedCourseRT.lvproj`, right-click **Build Specifications** → New → **Real-Time Application**
2. Set `Main.vi` as the startup/top-level VI
3. Set a destination directory on the target
4. **Build** — LabVIEW will offer to set it as the target's **startup application**; say yes
5. Run it directly from the Project Explorer to confirm it behaves exactly like the interactive version did

⚠️ *Once it's the startup app, it claims the target on every power-up — debugging afterward just means reconnecting properly, or temporarily disabling it. Normal part of working with embedded targets.*

✅ **Commit**: the updated `.lvproj`, now containing the build spec — `"Add Real-Time Application build spec, set as startup"`

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 2
## Power-cycle it, and watch

---

## 🔬 Science-fair round

Unplug your MyRIO. Plug it back in. **Don't open LabVIEW.**

- Watch it come back up on its own
- Point your `Receiver.vi` (or the free UDP tool) at its IP — data should already be flowing
- Tilt it, shake it — your team's algorithm should still react exactly as before

Walk around, watch everyone else's boot up too — this **is** the demo.

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 3
## The flight recorder: log it, then go find it

---

## ✈️ Why this matters now

The app runs headless now — nobody has to be watching the UDP broadcast for anything to be captured. The Logbook is no longer a nice-to-have, it's your **black box flight recorder**: you don't watch it live, but when you want to know what happened, it's there.

---

## 🛠️ One line first: log the boot itself

In `Main.vi`, right at startup — before anything else runs — send `{"Log", "Application started"}` to Logbook.

Every single boot now writes itself into its own record.

Rebuild the `.rtexe` (Work 1's steps), redeploy, reboot once more.

---

## 🛠️ Go find it — no supervision screen required

1. Let the MyRIO run a minute, tilt it a couple of times to generate some events
2. Open **MobaXterm** (or any SSH/SFTP client), connect to the MyRIO's IP
3. Navigate to the log file's location on the target's filesystem, open `logbook.txt`
4. Confirm: the startup entry is there, plus everything since — a full record, entirely independent of whether anyone was watching the live broadcast

✅ **Commit**: `"Log application startup"`, merge locally, push

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 📖 Wrap-up
## Docs, tag, ship it

---

## Tags vs. branches, one more time

A **branch** moves — every commit shifts where it points.

A **tag** doesn't — a fixed label on one specific commit, forever. Exactly what you want for "this is what I'm submitting."

---

## 🛠️ Close it out

1. Add a short **"Architecture"** section to `README.md` — not a build tutorial *(any LabVIEW dev already knows how to build and deploy)* — just: what's the startup VI, and which modules does it launch and why
2. Tag the current commit — `v1.0` — via Fork
3. Push the tag

✅ **Checkpoint**: `main` clean, tagged, and the README orients a stranger in thirty seconds

---

## ✍️ One last retrospective

Three bullets, committed as `RETROSPECTIVE.md`:

- What went well?
- What was genuinely hard, and why?
- If you started again tomorrow, what would you do differently?

---

## 📦 Deliverables today

- A built, working `.rtexe`, set as the MyRIO's startup application
- A successful power-cycle test with zero PC connection
- A startup log entry, confirmed by reading `logbook.txt` remotely
- `README.md` (Architecture section), `RETROSPECTIVE.md`, a pushed `v1.0` tag

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

## Closing

We opened this course pretending to run a burger joint. Six sessions later, underneath the story, we'd built a real-time acquisition, processing, and broadcast system — one message contract, applied patiently, module after module.

And today, for the first time, **nobody needs to believe that on faith.** Unplug it. Plug it back in. It's just... running.

*Not the burgers. The discipline. And now, the proof.*
