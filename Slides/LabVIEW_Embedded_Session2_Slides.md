---
marp: true
theme: default
paginate: true
size: 16:9
header: '**LabVIEW Embedded** · Session 2'
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

# 2️⃣ Session 2
## Real modules, error handling, first branches

### Get Order becomes real, and LV-King starts a logbook

---

## Today's plan *(relative to your session start)*

| Elapsed | Block |
|---|---|
| 0:00–0:10 | Welcome, recap |
| 0:10–0:50 | Concept 1 — Real modules, and talking back to your caller |
| 0:50–1:35 | Work 1 — Build Get Order as a real module |
| 1:35–1:50 | Break |
| 1:50–2:15 | Concept 2 — Errors that never disappear silently |
| 2:15–3:50 | Work 2 — Logbook, dispatch, rush alert |
| 3:50–4:00 | Merge check |

---

## 🔁 Recap

Last week, **Cooker** was a single VI listening to three commands.

Get Order was just a bare loop reading HMI buttons — not a module at all.

Today: **Get Order** becomes a **real module**, LV-King opens a **logbook**, and nothing is ever allowed to fail silently.

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🧩 Concept 1
## Real modules, and talking back to your caller

---

## 🛠️ Build the template once, reuse it forever

A single reference VI, `/sandbox/module_template/Handler.vi`:

- While Loop, `Dequeue Element`, and `Release Queue`
- Case structure, placeholder cluster through shift registers *(the module's private data, more on this shortly)*
- Generic `"Stop"` case: release the queue, exit the loop

---

## 🛠️ Build the template (2): a module can also talk back

A module always has a manager. It receives orders from it, and can talk back to it. It therefore needs:

- its **own queue**, to listen to the manager
- the **manager's queue**, to send information back

⇨ Add both queue controls to the template.

---

## 🛠️ Build the template (3): payload becomes Variant

`Message.ctl = {command: string, payload: variant}`

- A plain string was fine for a quantity — awkward for an error, or a more complex, structured piece of data
- Any type goes in now, cast back out with a type constant
- The `command` string tells the receiver what type to expect
  → **write that down** whenever you define a new message

---

## 🛠️ Build the template (4): unit test

Create a **unit tester**, `/sandbox/module_template/Module Tester.vi`:

- Obtain queues for both the self and manager inputs
- Drop the template Handler in parallel with a While Loop
- In that loop, dequeue the manager queue — this loop simulates the manager module. *Tip: use a 200 ms dequeue timeout.*
- When the loop stops (button click), send a `"Stop"` message to the template Handler

Run the tester, stop the loop, and check that everything stops properly.

✅ **Commit here** — module template

---

## 🌿 Git branches, the Fork way

A new feature, a code fix — that's a Git branch!

1. Branch button → name it (`feature/get-order-module`) → create
2. Work, commit along the way
3. Done? → **merge** into `main` → **push**

**Golden rule**: `main` must always be runnable and demoable.

---

## 📐 The recipe for a new module

1. Create `/src/Modules/MyModule`
2. Create an `.lvlib` in it
3. Copy `Handler.vi` from the template into the folder, then add it to the `.lvlib`
4. In the `.lvlib`, create the private data cluster, and use it to replace the placeholder in the Handler

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 1
## Build Main and Get Order as real modules

---

## 🛠️ Create the two modules, following the recipe

*Cooker is the manager of the restaurant — from now on, we'll call the module `Main`.*

Adapt Main's Handler, since it plays a special role — the app's manager:

- Its Handler **is** the top-level application VI
- Obtain every queue directly inside the handler, and drop Get Order's Handler inside it, running in parallel
- Find a mechanism to stop Main properly, and confirm the app can run and stop cleanly

---

## 📜 Get Order's contract

**Can send** to Main: `{"Cook burgers", quantity}`, `{"Cook a meal", ""}`, `{"Stop", ""}`

**Can receive** from Main: only `{"Stop", ""}` — nothing else for now.

---

## 🛠️ Build it

1. `GetOrder.lvlib`: `Handler.vi` — build your HMI (buttons and controls to order food)
2. Use the dequeue timeout to poll your buttons and send the matching command to Main
3. VI Properties → **"Show front panel when called"**
4. In Main: add a case for each command, and display the queue's current load

✅ **Commit**: `"Base modules"`, merge and push

---

## ⭐ Bonus, if time allows

Main can disable Get Order once too many orders are queued.

**Get Order's contract, updated** — **can receive** from Main: `{"Activate", true/false}`, enabling or disabling the front panel's controls.

When dequeuing an order, Main checks its own queue load: above the limit → send `Activate(false)`; back below → send `Activate(true)`. Be careful not to send it on every single message!

✅ **Commit**: `"Real bidirectional contract"`, merge and push

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🧩 Concept 2
## Errors that never disappear silently

---

## 🍔 The steak on the floor, formalized

The cook who drops a steak and serves it anyway without a word, versus the one who writes it down.

**Cooking will sometimes fail, and it must say so — as a message.**

---

## 📖 One logbook, everything worth remembering

Logbook becomes our **single logging destination** — every cook, every error, every rush alert.

**Why a module, and not a plain sub-VI?**

Because **file I/O is slow and non-deterministic** — it should never happen inside a time-critical module. Main and Get Order already have plenty to do; we hire a new employee just for this job.

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

# 🛠️ Work 2
## Logbook, dispatch, rush alert

---

## Step 1 — build Logbook, tested before it's even wired in

- `Logbook.lvlib` from the template: `{"Log", <LogEntry>}` → append to file
- `LogEntry.ctl = {timestamp: DBL, kind: string, text: string}` — the typed payload every `"Log"` message carries
- **Unit test inside the library**: `Test.vi` — manual queue, no `Main.vi` involved

✅ **Commit here** — built & tested, *not yet wired in*. A good branch is several small commits, not one at the end.

---

## Step 2 — make cooking occasionally fail, log it

- `Main.vi`: obtain Logbook's queue, run its handler in parallel
- Extend the shutdown broadcast to reach it too
- `"Cook a meal"` gets a ~15% simulated failure (a custom LabVIEW error)
- On failure: send `{"Log", <error as Variant>}` directly to Logbook
- On every successful cook: log it too — Logbook becomes our full record

✅ Validate, commit, merge into `main`, push

---

## ⭐ Bonus, if time allows: rush alert detection

- Add a rolling window of recent requests to Main's own private data
- On every cook request: update the window, log an alert — and disable Get Order — if there are more than 15 requests in the last 30 seconds

---

## 📦 Deliverables today

- `GetOrder.lvlib` — real bidirectional module, buttons that grey out
- `Logbook.lvlib` (with `Test.vi`)
- `/logs/logbook.txt`: cooks, errors, at least one rush alert
- Feature branch(es) merged locally, pushed

---

<!-- _class: lead -->
<!-- _backgroundColor: #1D3557 -->
<!-- _color: white -->

## 🔜 Next week

Main has quietly been doing two jobs since day one.

*And it turns out we haven't been running a restaurant this whole time...*
