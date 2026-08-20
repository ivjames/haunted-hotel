# Haunted Hotel

You arrive at a haunted hotel. The only way out is up: thirteen floors to
the roof, with a flashlight and thirteen points of sanity.

A father–daughter game design, circa 2011. HTML5 canvas, one file, no
dependencies, no build.

## How it plays

- **Climb.** Each floor is a corridor; find the stairs and take them up.
  Floor 13's door leads to the roof — reach it and you win.
- **Ghosts.** They drift toward you. Every real ghost that touches you
  costs **one sanity point**. You have **thirteen**. At zero you go
  completely insane, and the hotel keeps you.
- **Flashlight.** Hold the beam on a real ghost and it burns until it
  dies. Dead ghosts leave **glowing orbs** that recharge the flashlight;
  sometimes you just find orbs lying around. The battery drains while the
  light is on — an empty flashlight is a very bad time.
- **Fake ghosts.** As your sanity slips you begin to see ghosts that are
  not there. They look like the real thing, but they can't hurt you — and
  under the flashlight they simply **vanish** instead of burning, wasting
  your battery and your nerve. The more insane you are, the more of them
  you see.

## Controls

| Action | Keyboard / mouse | Touch |
|---|---|---|
| Move | ← / → or A / D | corner arrow buttons |
| Flashlight | hold mouse button (aims at cursor) or SPACE | touch and hold anywhere else (aims at finger) |

## Running it

It's one `index.html`. Open it in a browser, or serve the directory with
anything static. Production runs at `haunted.lab980.com` — see
[DEPLOY.md](DEPLOY.md).
