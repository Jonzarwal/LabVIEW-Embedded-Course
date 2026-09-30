---
marp: true
theme: default
paginate: true
size: 16:9
header: '**LabVIEW Embedded** · Session 5'
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

# 5️⃣ Session 5
## Real hardware, for real

### Deploying to MyRIO, and finally reading a real sensor

---

## ⚠️ Before today

- LabVIEW Real-Time module installed on every machine
- MyRIO drivers/software installed, unit connected and recognized
- Everyone has their own MyRIO in hand

*(logistics to sort out ahead of time — not something to debug live during the 4 hours)*

---

## Today's plan *(relative to your session start)*

| Elapsed | Block |
|---|---|
| 0:00–0:10 | Welcome, recap |
| 0:10–0:45 | Concept 1 — RT targets and deployment |
| 0:45–1:45 | Work 1 — Deploy Main to the MyRIO |
| 1:45–2:00 | Break |
| 2:00–2:30 | Concept 2 — The onboard accelerometer |
| 2:30–3:30 | Work 2 — Swap Acquisition for the real sensor |
| 3:30–4:00 | Work 3 — Full test, on real hardware |

---

## 🔁 Recap

Everything so far has run on your PC, fed by a simulated accelerometer.

Today, the exact same app moves onto **real hardware**, and starts reading **real motion**.

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🧩 Concept 1
## RT targets and deployment

---

## 🍟 Remember the automated fryer?

Session 3: an employee juggling ten tasks (Windows) vs. an automated fryer built for one guarantee (LabVIEW Real-Time).

**Today you stop talking about the fryer, and start deploying onto it.**

---

## 🖥️ Two projects, one set of source files

Adding a Real-Time target straight into your existing desktop project works for simple VIs — but with several `.lvlib` modules like ours, mixing a Windows target and an RT target in **one** project causes real trouble: locked libraries, target conflicts.

**Cleaner split**: a brand-new project, e.g. `LVEmbeddedCourseRT.lvproj`, dedicated to the MyRIO target — pointing at the exact same `/src` files. Nothing is duplicated, you just get two clean project trees instead of one messy one.

---

## What actually changes when you deploy

- Code gets compiled and pushed **onto the MyRIO's own OS**, not just interpreted on your PC
- It keeps running on the target even if you close LabVIEW on your laptop *(once truly deployed — more on this next session)*
- File paths, network interfaces — everything now belongs to **the target**, not your development machine

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 1
## Deploy Main to the MyRIO

---

## 🛠️ Steps

1. Create a new project, `LVEmbeddedCourseRT.lvproj`, next to the original
2. Add your MyRIO as a **Real-Time target** inside this new project
3. Add `Main.vi` and its dependent modules to the RT target — same `/src` files, nothing duplicated
4. Run it **interactively** on the target — same behaviour as on your PC, just physically executing on the MyRIO now
5. Confirm Logbook and Broadcast still work — **but check where**

---

## ⚠️ Watch out: paths and network moved with the code

- Logbook's log file now lives on the **MyRIO's own filesystem**, not your PC's
- Broadcast now sends from the **MyRIO's network interface**
- Your receiver (or the free UDP tool) needs to point at the **MyRIO's IP**, not `127.0.0.1` anymore

✅ **Commit**: `"Deploy Main to MyRIO target (still simulated data)"`

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🧩 Concept 2
## The onboard accelerometer

---

## What it actually measures

The MyRIO has a **3-axis accelerometer built in** — no wiring, no external board.

- Output: `x`, `y`, `z`, in **g**
- At rest, it reads gravity — exactly the vector our Session 3 tilt discussion was built on
- Some sensor noise, unlike our clean sine + noise simulation — expect it to look messier

---

## Reading it in LabVIEW

A single Express VI on the MyRIO palette gives you `x`, `y`, `z` directly — no register-level work needed for this course.

**The output shape is exactly `AccelSample.ctl`.** That's not a coincidence — it's why we picked that typedef back in Session 3.

---

## 🧩 One source, two targets

A **Conditional Disable Structure** compiles different code depending on the target — like an `#ifdef`, built into LabVIEW.

- One case compiles only when building for **Windows** *(the simulated sample)*
- One case compiles only when building for **Real-Time** *(the real accelerometer read)*

The disabled branch isn't just skipped, it's **stripped out at compile time** — no RT-only node ever needs to resolve on your desktop project, and vice versa.

Same file, same contract, two targets, automatically.

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 2
## Swap Acquisition for the real sensor

---

## 🛠️ The swap

Inside `Acquisition.lvlib`'s **hidden helper loop** — and *only* there:

1. Wrap the sample-generation code in a **Conditional Disable Structure**
2. **Windows case**: keep the simulated sine + noise
3. **Real-Time case**: the onboard accelerometer read, packed into `AccelSample.ctl`
4. Everything else — command loop, batching, Local Variables, contract — untouched. The same file now works, unchanged, in both projects.

If this is the only thing you had to change, the contract held.

---

## 🔬 Test it

- Run your `Acquisition` `Module Tester.vi` first, standalone, on the target — confirm real batches of 10 real samples arrive
- Then re-run the full app: `Main` + `Acquisition` + `Logbook` + `Broadcast`, all on the MyRIO

✅ **Commit**: `"Acquisition reads the real onboard accelerometer"`

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 3
## Full test, on real hardware

---

## 🔬 Make it move

- Hold the MyRIO flat, then tilt it — watch your team's algorithm react to **real motion**
- Group 1: shake it and watch the alarm trip
- Group 2: tilt it and watch your value change — does your method hold up on a real, noisy signal?
- Watch it over UDP: your `Receiver.vi`, or last session's free tool, now pointed at the **MyRIO's IP**

✅ **Commit**, merge locally into `main`, push

---

## 📦 Deliverables today

- `LVEmbeddedCourseRT.lvproj`, dedicated to the MyRIO target, referencing the same `/src`
- `Main.vi` and every module deployed and running interactively on the MyRIO
- `Acquisition.lvlib` reading the real onboard accelerometer via a Conditional Disable Structure — same file works on both projects
- A live test against real motion, observed end to end over the network

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

## 🔜 Next session — the final boss

Everything you built runs on the MyRIO now, but only while LabVIEW is watching over it.

*Next: a real standalone `.rtexe` — the app boots and runs on its own, no PC required.*
