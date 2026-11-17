# Petri Dish

A mixed-reality tabletop god-game for **Meta Quest 3**, in the browser, with WebXR and three.js. Played with bare hands.
A [NXT/WebXR](https://www.nxtwebxr.com) production.

Put a Petri dish on your real desk. The dish flashes a sequence of colours; answer it, drop by drop, with glass droppers of glowing stain. Get it wrong and the agar dies, comically. Get it right and something wakes up. Watch it under the microscope as cells become colonies, colonies become a town, and the town grows big enough to see without the microscope. Then they notice you.

Built on the die Hände skeleton (`nxt-thing-hand`): session setup, passthrough dimmer, hand tracking and hand occluders, the synthesized audio bus, the `?sim` desktop preview. It is its own piece and its own repo.

## Status

Direction stage. Nothing is built yet beyond the scaffold. See `DIRECTION.md` for the short version and `docs/RESEARCH_AND_DIRECTION.md` for the research, art rules and plan.

## How it is made

- `index.html`: the whole piece, one file, no build step (to come).
- `assets/hand-left.glb`, `assets/hand-right.glb`: the generic WebXR hand meshes from [`@webxr-input-profiles/assets`](https://github.com/immersive-web/webxr-input-profiles) (MIT), drawn depth-only so your real hands pass in front of the lab.

## Desktop preview

`python3 -m http.server 8781`, then open `http://localhost:8781/?sim` (to come).
