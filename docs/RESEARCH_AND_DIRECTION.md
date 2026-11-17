# Petri Dish: research and direction, v0

A mixed-reality tabletop god-game for Meta Quest 3, played with bare hands. Put a Petri dish on your real desk, answer the colours it flashes with glowing droppers, and raise a tiny civilisation that ends up knowing who made it.

Status: research and direction only. Nothing beyond a scaffold has been built. Morten reacts to this first.
Date: 17 Nov 2026. Skeleton: `nxt-thing-hand` (die Hände). This is not a competition entry, so code from die Hände can be reused directly.

---

## 0. The decisions in one screen

| Topic | Decision |
|---|---|
| Look | A 1950s American teaching lab: honey agar, seafoam and cream enamel, oxblood Bakelite, a black crinkle-finish microscope, lit by one warm lamp. The beings come from the **Bell System Science Series** (Capra, 1956 to 1957): the lab is drawn as live action, and the life inside it is drawn as a mid-century educational cartoon. |
| Glass | No `transmission`. A custom fake-glass shader: Fresnel rim, a warm procedural environment reflection, edge thickness tint, premultiplied blending over passthrough. |
| Post-processing | None full-screen. Grain, vignette and warm tint go on a premultiplied "room dimmer" sphere, the technique die Hände already uses. |
| Pipette | **Sticky pick, pinch to squeeze.** Pinch near a dropper and it jumps to your hand and stays there without holding the pinch. Each pinch after that squeezes the bulb and makes one drop. Bring it back to the rack to put it down. |
| Drops | An SDF impostor (two spheres with a smooth union, for the neck as it pinches off), stretched along velocity, emissive core plus additive halo. No metaball fields. |
| Colour mixing | Not a mechanic. Each drop is one signal. Colours only blend visually in the agar, and the mix decides what the colony grows into. |
| Growth | Six readable stages: agar, sparks, cells, colonies, structures, beings, then citizens you can see without the microscope. Each stage changes silhouette, colour, density and motion together. |
| Microscope | Swing it over the dish and lean into the eyepiece. A circular, stereo, 40x bright-field view of the same simulation fades in. |
| Voice | **No word recognition.** `getUserMedia` plus WebAudio analysis: loudness, onsets and pitch. Cheer when it is loud, hum and sway along with your pitch, hush when you go quiet, startle at a clap. The game works fully without the mic. |
| Placement | Plane detection (a `table` plane) first, then hit-test (depth-based on Quest 3/3S), then die Hände's head-relative fallback. All optional features, never required. |
| Occlusion | die Hände's depth-only hand meshes. No depth-sensing API in v1. |
| Stack | One `index.html`, no build step, three.js r150.1 from an import map, synthesized audio. The same as die Hände. |

---

## 1. What die Hände gives us (read in full: README, index.html, assets)

These things are already proven live on nxtwebxr.com and can be copied as they are:

- **Session:** `navigator.xr.requestSession('immersive-ar', { requiredFeatures: ['local-floor'], optionalFeatures: ['hand-tracking'] })`, set with `renderer.xr.setSession`. Alpha renderer with clear colour 0, ACES tone mapping, sRGB output, PCF shadows. die Hände does **not** use hit-test, plane detection, anchors or depth sensing. It places the bench in front of the head (`placeLab(headPos, forward)`).
- **Hands:** `renderer.xr.getHand(i)` and its 25 joints, brought into lab space (`toLab`). Per-finger openness, palm velocity, a tracking-loss timer. There is a controller fallback (trigger and grip curl the fingers) and the generic WebXR hand GLBs (MIT, `@webxr-input-profiles/assets`). These are already copied into `assets/`.
- **Hand occlusion:** your real hands are drawn as skinned meshes that write depth only, so they pass in front of virtual objects and cast shadows. This is better than the Quest depth map for hands (see section 6).
- **Room dimmer:** a sphere round the head with premultiplied alpha. RGB adds light, alpha dims passthrough. It already has world-locked film grain, so both eyes agree. This is our grain, vignette and warm tint, with no post pass.
- **Audio:** an `AudioContext` with a compressed bus, an HRTF panner per sound, a generated reverb, every effect synthesized, and one looping soundtrack through a limiter.
- **Shell:** a landing page where the poster is the live scene, the NXT/WebXR studio footer (`nxtwebxr.png`, monochrome, "A NXT/WebXR production"), a canonical URL and OG tags, `?sim` desktop preview with the mouse as a hand, `window.__T` test hooks with `go(beat)`, a `CFG` block at the top of the script, and pre-compiled shaders so nothing hitches.
- **Conventions:** README tone (prose, the beats, "How it is made", "Desktop preview"), no en or em dashes in player-facing copy, `.nojekyll`, GitHub Pages mounted under nxtwebxr.com.

---

## 2. Research: the 1950s American lab look

### What the period actually looked like

- **Glass:** borosilicate (Pyrex, Kimax) was the standard lab glass by mid-century. Corning's own 1950s organic chemistry kit guide says the PYREX mark had been "the recognized standard" for almost half a century. Kits came in fitted wooden boxes ([Science Museum Group, semi-micro set c. 1950](https://collection.sciencemuseumgroup.org.uk/objects/co14912); [Corning organic chem guide](https://web.mst.edu/lib-circ/files/chem/CHEM2229-FORMERLY-CHEM228-Bone/CORNINGORGANICCHEM_308.pdf)). Period glass carries white or amber **printed graduations** and a small maker's mark, and it is never spotless: there is a faint green or straw tint at thick edges.
- **Chemistry sets:** the 1950s were the heyday of chemistry sets. Gilbert and Chemcraft sold them in lithographed tin cases, typically blue, red or orange-blue, with illustrated lids ([Science History Institute, "Chemistry at Play"](https://www.sciencehistory.org/stories/magazine/chemistry-at-play/); [Gilbert U-238 Atomic Energy Lab, 1950](https://orau.org/health-physics-museum/collection/toys/gilbert-u-238-atomic-energy-lab.html); [The Henry Ford, Atomic Energy Lab](https://www.thehenryford.org/collections/explore/artifact/335518)). From these we take the hopeful, atomic-age graphics, not the toy scale.
- **Microscopes:** mid-century American instruments moved from lacquered brass to **black or grey crinkle enamel with chrome** fittings and black Bakelite knobs, in fitted wooden cases ([Chairish, mid-century Rochester microscope](https://www.chairish.com/product/35749958/vintage-bausch-and-lomb-microscope-in-original-locking-wood-case); [Ingenium, grey enamel instrument](https://collections.ingeniumcanada.org/en/id/1988.1202.001)). Projection microscopes existed too ([B&L carbon-arc projection microscope](https://chsi.emuseum.com/objects/2302/bl-carbonarc-projection-microscope)), and they are a fallback idea for section 5.
- **Interiors and photography:** Bell Labs and other research labs of the period were photographed in hard, single-source light with deep shadows and a warm cast ([AT&T archives essay](https://soundandscience.net/contributor-essays/the-att-archives-and-history-center/); [NRAO: Bell Labs c. 1950](https://www.nrao.edu/archives/items/show/28465); [Feininger, Bell Labs 1943](https://artsandculture.google.com/asset/modern-laboratory/FAEc0MPRnoCEng)).
- **Educational film:** the **Bell System Science Series** (*Our Mr. Sun* 1956, *Hemo the Magnificent* 1957, Frank Capra, animation directed by William T. Hurtz) cut a live-action lab against animated characters for the invisible world ([Wikipedia: Hemo](https://en.wikipedia.org/wiki/Hemo_the_Magnificent); [Our Mr. Sun](https://www.wikipedia.org/wiki/Our_Mr._Sun); [overview](https://brewminate.com/the-bell-system-science-series-advancing-the-public-understanding-of-science-1956-1964/)). This is the key to the whole art direction. The lab is real, solid and slightly worn. The life is graphic, simple and warm, and the two meet at the glass.
- **Colour:** genuine mid-century palettes are seafoam, mint, light turquoise and butter yellow, on cream or bone, and almost never pure white ([Dunn-Edwards, palettes by decade](https://www.dunnedwards.com/?p=30150); [Made Good Designs](https://madegooddesigns.com/vintage-color-palette/)). Published hex values disagree, so the palette below is our own, tuned for passthrough.

### Turning that into three.js on Quest

- **Glass without transmission.** `MeshPhysicalMaterial.transmission` costs an extra pass that re-draws the opaques. Users report frame rates halving, and there are known stereo and Quest bugs ([three.js forum: transmission cost](https://discourse.threejs.org/t/transmission-effect-pbr-material-performance/59274); [stereo issue](https://lightrun.com/answers/mrdoob-three-js-meshphysicalmaterial-transmission-doesnt-work-correctly-in-webxr-immersive-mode); [8th Wall: Quest 3 issues](https://forum.8thwall.com/t/three-js-project-on-meta-quest-3/6698)). Meta also notes Quest is usually fill-rate bound and transparency means overdraw ([Meta WebXR perf](https://developers.meta.com/horizon/documentation/web/webxr-perf-bp/)). Our glass shader is therefore:
  1. Schlick Fresnel (F0 = 0.04) times a reflection from a **procedural warm lab environment**. This is a small equirect we paint ourselves: a lamp hot spot, a long cool window strip and a cream ceiling, run through PMREM once. The bright strips give glass its read.
  2. **Thickness tint:** `1 - |dot(N, V)|` drives a faint straw-green (`#D8E3C0`) at silhouettes and on the rolled rim, the way thick borosilicate edges look.
  3. **Premultiplied output:** RGB adds reflections, alpha dims the room only a little at the rim. Glass never makes a grey fog over the desk.
  4. **Fake refraction:** liquids and agar inside glass get a small view-dependent offset and scale in their vertex shader, so they swim slightly as you move. There is no screen-space grab.
  5. Two draws per vessel, back faces then front faces, sorted by hand. Interior surfaces (the meniscus, the agar top) are their own opaque or premultiplied meshes.
- **Enamel:** `MeshStandardMaterial`, roughness 0.35 to 0.45, a low-frequency orange-peel normal from noise, and **chips** at high-curvature edges showing dark steel (a curvature mask baked into vertex colours at build time).
- **Bakelite:** standard material, roughness 0.22, a marbled swirl from domain-warped noise between two oxbloods, slight clearcoat look from a second sharp specular lobe added in `onBeforeCompile` (no `MeshPhysicalMaterial`).
- **Chrome and brass:** metalness 1. Roughness 0.08 for chrome, 0.3 for brass with fbm tarnish. Both depend on the warm environment map, which is why it gets art-directed rather than generated.
- **Rubber bulbs:** soft red-brown with a powdery bloom (a sheen-like rim term). They deform with your pinch (section 4).
- **Typography on props:** canvas textures drawn from Google Fonts after `document.fonts.load`, as die Hände does it. Graduations are printed white enamel on glass (alpha-tested). Labels are cream gummed paper with a red rule and typewritten contents.
- **Grain and vignette in passthrough:** we cannot touch the camera image, and a full-screen post pass in WebXR costs MSAA, foveation and a full extra fill. So grain, warm tint and the soft vignette ride on die Hände's dimmer sphere, world-locked. Keep it subtle: the room stays readable.

---

## 3. Art direction rules (non-negotiable)

### Palette

| Role | Name | Hex |
|---|---|---|
| Agar (lit) | Honey agar | `#E7B34E` |
| Agar (deep, through glass) | Amber agar | `#B9802B` |
| Equipment enamel | Seafoam | `#9DC8B3` |
| Equipment enamel, shade | Lab green | `#5F8C7B` |
| Enamel neutral | Cream enamel | `#EFE6CF` |
| Enamel neutral, shade | Bone | `#CFC2A2` |
| Knobs, rack ends, dish-lid handle | Oxblood Bakelite | `#5A1A17` / swirl `#3A0F0D` / lift `#7A2A22` |
| Wood (rack, microscope case) | Walnut | `#5B3A24` / grain `#3B2416` |
| Metal | Brass | `#B8913F` / aged `#8C6B2C` |
| Microscope body | Black crinkle | `#1C1B19` |
| Bulbs | India rubber | `#9A3E2A` |
| Labels | Gummed paper / ink / red rule | `#F4ECD6` / `#2A2620` / `#A3322A` |
| Key light | Incandescent | `#FFD7A0` (about 2800 K) |
| Rim light | Window | `#BFD9E6`, low |
| Stain 1 | Malachite green | `#46C27A` |
| Stain 2 | Eosin red | `#E0473F` |
| Stain 3 | Picric yellow | `#F2C43D` |
| Stain 4 (later) | Methylene blue | `#3E86D6` |
| Life glow | Spark | `#FFE6A8` (warm white, never cyan) |
| Death | Soot / mould / fuzz | `#2B2A26` / `#6F7560` / `#B9B8A6` |
| Microscope field | Bright field | `#F3E7C8` |

The stains are named after real stains a 1950s lab would have used. Each one also gets a **distinct luminance, glyph and note**, so red and green are never told apart by hue alone (about 8% of men are red-green colour-blind):

| Stain | Glyph on label and in flash | Note |
|---|---|---|
| Green | circle | C5 |
| Red | triangle | E5 |
| Yellow | square | G5 |
| Blue | diamond | B4 |

### Lighting

- One warm key: a gooseneck lamp clamped to the tray, the motivated source. It casts real shadows from the props onto the tray and onto a shadow-catcher on the desk.
- One faint cool rim from the "window" in the environment map, which mainly exists to put a line of light on every glass edge.
- The microscope has its own little lamp, and it pours light up through the dish when it is swung over.
- Life emits light. As the civilisation grows, the dish becomes a second light source and lights your hand from below.
- Passthrough dimmed lightly (about 0.3, against die Hände's 0.74) with a warm wash. This is a lab in daylight, not a horror stage.

### Typography

- **Jost** (a Futura stand-in, Google Fonts): decals, the maker's mark, landing-page headings. Futura is the face of mid-century American science graphics.
- **Oswald** (an Alternate Gothic revival): numbers, graduations, the dial on the microscope.
- **Courier Prime**: typed index cards and labels.
- **Yellowtail**, sparingly: a single script maker's name on the microscope arm.
- In-world makers are **fictitious** (for example "Haulik Optical Co., Rochester N.Y.", "PETRI-WARE"). No real trademarks (Pyrex, Kimax, Bausch & Lomb) on props.

### Do

- Lab first, life second. Every prop should look like it could be bought at a 1955 lab supply house: honest proportions, visible screws, wear on handled edges.
- Use **one** material vocabulary per prop: glass, enamel, Bakelite, chrome, brass, walnut, rubber. Seven materials across the whole set, no more.
- Let light carry the beauty: highlights on glass, glow from stains, the lamp pool.
- Animate with restraint: a few things move beautifully, the rest stays still.
- Life is simple and graphic: round forms, dot eyes, one colour accent per lineage. It is charming because of how it moves, not how detailed it is.
- Copy has humour, deadpan and sparing, in the voice of a 1950s educational narrator.

### Don't (anti-slop)

- No sci-fi lab: no holograms, hex grids, neon cyan, scanlines or UI floating in space.
- No cartoon-parody lab: no bubbling green flasks, mad-scientist props, lightning (that belongs to die Hände), or comic proportions on equipment.
- No glossy plastic, no pure white, no pure black, no saturated primaries outside the four stains.
- No generic cute mascots: no big shiny anime eyes, no kawaii blush, no blob-with-a-face that could be any mobile game.
- No bloom halo on everything, no lens flares, no chromatic aberration across the screen.
- No `transmission`, no post-processing pass, no stock HDRIs (the environment is painted for this room).
- No text walls. Instruction comes from the dish itself (flashes, glyphs, sound) and one typed card at most.
- No AI-generated textures or images. Everything is procedural or drawn in code.
- No en or em dashes in any player-facing copy.

---

## 4. Research: hand-tracked pipette interaction

### Findings

- **Meta's own grab thresholds** ([Meta grab specs](https://developers.meta.com/horizon/design/grab_specs/)): pinch starts at **1.6 cm** thumb-to-index, and stops at **2 cm above the closest distance reached, capped at 3 cm**. Hover enters at 5 cm and exits at 8 cm. Palm grab starts at curl 0.9 and releases at 0.85. Hysteresis is the core pattern everywhere ([dmnshd: WebXR hand tracking](https://dmnshd.gg/blog/webxr-hand-tracking)).
- System pinch on Quest fires WebXR `select`, but it releases on tracking loss, and it is not available per finger. We read the joints ourselves, as die Hände does.
- Tracking loss is normal: hands near a desk, one hand occluding the other, fast moves. Grabs must survive about 300 ms of loss.
- "Object moves to hand" snapping is one of Meta's recommended grab alignments.
- **Droplets:** SDF impostors with smooth union are the standard real-time route ([Codrops: droplet metaballs in three.js](https://tympanus.net/codrops/2025/06/09/how-to-create-interactive-droplet-like-metaballs-with-three-js-and-glsl/); [Codrops: liquid raymarching](https://tympanus.net/codrops/2024/07/15/how-to-create-a-liquid-raymarching-scene-using-three-js-shading-language/)). With only a handful of drops on screen, a two-sphere impostor on a small quad is cheaper than marching-cubes metaballs and gives better control of the neck.

### Design: "sticky pick, pinch to squeeze"

The hard problem is that a dropper needs two actions, hold and squeeze, and Quest hand tracking only reads one gesture reliably: the thumb-index pinch. Holding a pinch while making a second gesture is fragile. So **holding should not need a gesture at all**:

1. **Hover:** your pinch point (midway between thumb tip and index tip) within **6 cm** of a dropper bulb highlights it: a glint runs down the glass, the bulb lifts 3 mm, a soft tick. Exit at 9 cm. If more than one is in range, the nearest one wins, with a 1.5 cm advantage for the one already highlighted, so it doesn't flicker.
2. **Pick:** a pinch closes (start below **2.0 cm**, a little looser than Meta's 1.6 cm because our joint distances are noisier than the system pinch). The dropper **snaps to the hand** over 80 ms, with the bulb set between thumb and index. From then on it **stays attached without holding the pinch**.
3. **Carry:** the dropper hangs **tip-down from the pinch point like a pendulum** (70% toward vertical, 30% hand orientation, damped). You always drip straight down, and aiming is just moving your hand over the dish.
4. **Squeeze:** each new pinch (close below 2.0 cm, re-arm above min + 2 cm, capped at 3.5 cm) is **one drop**. Between pinches the bulb **squashes continuously** with your actual thumb-index distance, so it feels physical even though it is attached. One drop per press, never a stream: the puzzle needs discrete drops.
5. **Aim:** a faint stain-coloured spot on the agar shows where the drop will land. If the tip is within 4 cm outside the rim, the drop's fall is bent up to 2 cm toward the dish. Drops that miss land on the tray as a small splat. **A miss is never a fail.**
6. **Put down:** bring the dropper within 5 cm of its rack slot and it docks itself. Touch another dropper with it and they swap. Open your hand flat and hold it for 0.6 s and the dropper floats back to the rack. If tracking is lost for more than 2 s, it goes back on its own.
7. **Refill:** each dropper holds 8 drops. Dip the tip into its reagent bottle to draw it back up, with the bulb inflating and the liquid column rising.

A second mode to test early in the headset: **dwell to drip** (hold the tip over the dish for 0.6 s for a drop). It is less tactile, but it removes gesture risk completely. It could be an accessibility toggle.

### Liquid and droplet look

- **Inside the dropper:** an emissive liquid column with a **concave meniscus** (a cap mesh), a brighter rim line where liquid meets glass, and level that drops with each drop.
- **Forming:** a small bead grows at the tip as the bulb squashes, using a two-sphere SDF impostor (tip sphere plus hanging sphere, smooth union), and the neck thins until it pinches off.
- **Falling:** a single sphere impostor, stretched along velocity, with an emissive core, a Fresnel rim reflecting the environment, and an additive halo sprite. Glow reads well over passthrough.
- **Landing:** a small splash crown (four to six instanced beads), an expanding ring, and the stain **blooms into the agar**: it is injected into the colony simulation texture (section 5) and spreads.
- **Mixing:** no mixing mechanic. Drops in the agar blend visually, and the proportion of each stain over the whole game decides the civilisation's character (green builders, red movers, yellow lantern-makers, see section 5).

### The sequence puzzle

- The dish **flashes**: the agar lights in the stain colour with its glyph appearing in the light, plus its note on a vibraphone. Three stains, sequences of 3 rising to 6.
- **Order matters, timing does not** in v1. Dripping in rhythm with the dish's beat gives a small extra sparkle and nothing else.
- **Wrong colour = fail.** Taking longer than 10 s on one step = the dish replays the sequence once, then a second time-out fails it.
- A drop that misses the dish never counts against you.

---

## 5. Research and plan: growing a micro-civilisation

### What makes tiny growth readable elsewhere

- **Townscaper** ([how it works](https://www.gamedeveloper.com/game-platforms/how-townscaper-works-a-story-four-games-in-the-making)): pieces matter most where shapes meet (corners, junctions). Detail is a second pass laid on top of a settled structure. The big shapes are predictable, the small ones vary. Birds and other life just anchor to nearby geometry. Outlines stay crisp at any distance.
- **Tiny Glade** ([Deep Root Depths analysis](https://deeprootdepths.substack.com/p/tiny-glades); [80.lv interview](https://80.lv/articles/exclusive-tiny-glade-developers-discuss-bevy-proceduralism-publishers-cozy-games)): very few structure types (rectangular, round, wall, fence), and the procedural system does the tedious detail.
- **Spore** ([CGW, "Evolutionary Chain"](https://www.cgw.com/Publications/CGW/2008/Volume-31-Issue-10-Oct-2008-/Evolutionary-Chain.aspx); D.I.C.E. 2008 talk summaries): parts were made deliberately cartoony, abstract and simple so procedural assembly stays readable. The cell stage is a top-down tide pool, which is the same view as ours.
- **Pikmin** ([Nintendo, Ask the Developer](https://www.nintendo.com/th/interview/ampy/03.html); [Mechanics of Magic, Pikmin 3](https://mechanicsofmagic.com/2021/04/21/visual-design-of-games-cheese-or-font-pikmin-3/)): a leaf on the head made tiny creatures readable, and **colour** became the identity of each kind. Real objects convey scale.
- **Real colonies** ([Ben-Jacob, branching patterns](https://www3.nd.edu/~zxu2/acms60212-40212-S12/final_project/Bacterial_branching_E_Ben_Jacob.pdf); [Golding et al.](https://arxiv.org/pdf/cond-mat/9807088); [ring patterns](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5746235/)): bacteria on agar form branching fronts (from nutrient diffusion) and concentric rings (from switching between growth states). A reaction-diffusion texture gives us exactly this look cheaply.

### Readability budget

Quest 3 shows about 25 pixels per degree. At 50 cm, a **4 mm being is about 0.46 degrees, roughly 11 pixels.** That is enough for a silhouette and motion, not for a face. So naked-eye beings are **5 to 6 mm**, carry their own light, and read by **motion and colour first**. Faces only exist under the microscope. The dish is a **large 140 mm dish** (standard is 90 to 100 mm) so there is room for a town.

### Stages

Every stage changes **silhouette, colour, density and motion** together, so each one is obviously new.

| # | Stage | Seen how | Silhouette | Colour | Density | Motion | Unlocked by |
|---|---|---|---|---|---|---|---|
| 0 | **Sterile agar** | naked eye | flat honey disc, meniscus at the rim | honey | none | none | placing the dish |
| 1 | **Sparks** | naked eye and scope | points of light | warm white, tinted by the stain just dripped | 5 to 20 | Brownian jitter, slow pulse | first sequence (3) |
| 2 | **Cells** | scope (naked eye: shimmer) | round glassy cells with a dark phase-contrast rim and a nucleus; dividing in two | stain tints on clear | 50 to 300 | division, cilia beat, drift | sequence (4) |
| 3 | **Colonies** | naked eye as patterns, scope for detail | branching fronts and concentric rings in the agar | lineage colours, blended where they meet | a field | the front grows slowly, rings pulse | sequence (4) |
| 4 | **Structures** | scope, faintly naked eye | small domes that merge; arches where two clusters meet; a spire at each cluster centre | translucent, glowing from inside | 10 to 40 buildings | they grow upward over seconds | sequence (5) |
| 5 | **Beings** | scope (faces), naked eye as moving lights | bean-shaped body, one tuft crest in the lineage colour (the Pikmin leaf), two dot eyes | cream body, coloured crest | 20 to 60 | waddle, gather, carry things | sequence (5, in rhythm) |
| 6 | **Citizens** | naked eye | the same beings at 5 to 6 mm, with lanterns; their town is **Googie / atomic-age** architecture: boomerang roofs, starbursts, antennae | warm lamplight in the dish | 60 to 150 | follow your hand, cheer, build | sequence (6) |
| 7 | **Recognition** | naked eye | they build a monument: **your hand**, in their architecture | gold lamplight | everyone | gather, celebrate, react to voice | reaching stage 6 |

The idea that makes the arc satisfying: **the civilisation grows into the lab's own era.** Their final architecture is 1950s atomic-age modern, they make lanterns like the lamp above them, and the monument is your hand. You are their weather and their god.

### Lineages from the stains

- **Green: builders.** More domes, gardens in enclosed courtyards (a Townscaper-style rule: gardens only grow where buildings enclose a space).
- **Red: movers.** Faster beings, paths worn into the agar, little boats in the meniscus.
- **Yellow: lantern-makers.** More light, lanterns on spires, festivals at night.

The town reflects your drip history. Two runs look different without needing a complicated rule set.

### How it is drawn (cheaply)

- **Colony field:** one Gray-Scott reaction-diffusion texture (256 x 256, ping-pong, 30 Hz) on the agar, seeded by drops. It gives branching and rings for free, and it also drives where structures may grow.
- **Cells and sparks:** instanced quads with the same SDF impostor shader as the droplets. Division is the two-sphere smooth union in reverse.
- **Structures:** instanced meshes from a small kit (dome, arch, spire, bridge, garden patch, lantern, Googie roof, antenna), placed by rules on the colony field. Detail is a second pass, as in Townscaper.
- **Beings:** one instanced mesh with no skeleton, animated in the vertex shader (bob, lean, crest sway, squash on landing). Matte "gelatine" shading with a soft wrap light and a rim, no toon outlines.

### The microscope

- A black crinkle-finish monocular with chrome, a Bakelite focus knob, a walnut case, a little lamp.
- **Use:** grab its arm (sticky pick, as for droppers) and swing it over the dish. It docks. Lean your eye to the eyepiece: within 12 cm, the view crossfades into a **circular field** (stencil disc in front of the eyes) showing the dish simulation at **40x**, in stereo, in the **bright-field look**: warm cream light, glassy cells with dark rims, a little dust, a slight chromatic fringe at the field edge. Turn the knob to focus (a depth-of-field blur on the micro layer only).
- It is the same simulation, rendered a second time at scale. No separate world, so there is nothing to keep in sync.
- **Fallback if leaning in is uncomfortable:** a projection microscope that throws the field onto a round enamel screen on the tray.

### Hand and god interactions (stage 6 onward)

- Citizens **look at your nearest fingertip**, gather beneath it when it is still over the dish for 1 s, and scatter if it comes down fast.
- Your hand **shades the dish** (the shadow technique from die Hände). They react to the eclipse.
- Touch the agar with a fingertip and one climbs on. Lift it slowly and it rides. Tip your finger and it slides back down.
- A failed sequence at this stage is a **blight**, not a wipe: sooty mould takes a sector, citizens panic comically, the next success clears it.

### Failure (before there is life)

The dish **dies comically**: the agar curdles grey-green, fuzzy mould races out from the wrong drop, a small puff of smoke, a sad descending slide-whistle. Then a fresh dish slides off a stack of clean dishes, and the dead one goes into an enamel pedal bin with a clang.

---

## 6. Research: platform support (Quest Browser, Nov 2026)

| Feature | Status | Use in Petri Dish |
|---|---|---|
| `immersive-ar` passthrough | Supported, colour on Quest 3/3S ([Meta MR docs](https://developers.meta.com/horizon/documentation/web/webxr-mixed-reality/)) | Yes, as die Hände |
| `hand-tracking` | Supported, 25 joints ([dmnshd](https://dmnshd.gg/blog/webxr-hand-tracking)) | Yes, as die Hände |
| `plane-detection` | Supported. Planes are horizontal or vertical rectangles. `detectedPlanes` fills in **2 to 3 s** after session start. `initiateRoomCapture` once per session ([Meta MR docs](https://developers.meta.com/horizon/documentation/web/webxr-mixed-reality/)) | Optional. Find a table-height horizontal plane and suggest it. |
| `hit-test` | Since Browser 40.4 (Oct 2025) it uses the **Depth API, not the scene mesh**, on Quest 3/3S. **No room scan needed** ([UploadVR](https://uploadvr.com/quest-browser-depth-api-webxr-hit-testing-instant-placement)). Oct 2026 adds experimental HTML-in-Canvas hit testing ([release notes](https://developers.meta.com/horizon/documentation/web/browser-release-notes/)) | Optional. Placement reticle from the hand ray. |
| `anchors` | Supported, up to 8 persistent per site ([Meta MR docs](https://developers.meta.com/horizon/documentation/web/webxr-mixed-reality/)) | Optional. Anchor the tray so it does not drift. Persistent anchor if we keep save games (open question). |
| `depth-sensing` | GPU-optimized only, depth camera at about 30 fps, low resolution with gaps at edges ([UploadVR](https://www.uploadvr.com/quest-3-depth-api-mixed-reality-dynamic-occlusion/); [three.js forum](https://discourse.threejs.org/t/webxr-depth-sensing-api-dont-work-in-three-js-r168/70675); [IWSDK guide](https://developers.meta.com/horizon/documentation/web/iwsdk-guide-depth-sensing/)). three.js support needs a much newer version than r150. Apr 2026 adds experimental depth projection ([release notes](https://developers.meta.com/horizon/documentation/web/browser-release-notes/)) | **Not in v1.** The tray sits on the desk and the only moving occluders are hands, which our hand meshes handle more precisely. |
| WebGPU in WebXR | Added in Browser 156.0 (13 Nov 2026) ([release notes](https://developers.meta.com/horizon/documentation/web/browser-release-notes/)) | Not now. Too new. |
| Camera access | Not exposed to pages ([Meta MR docs](https://developers.meta.com/horizon/documentation/web/webxr-mixed-reality/)) | No. Glass reflects our painted environment. |

The browser release notes from Feb to Nov 2026 list nothing new for plane detection, anchors or hand tracking, so the older docs still stand. Everything is requested as **optional** so a missing room setup never blocks a session.

---

## 7. Research and recommendation: "citizens react to voice"

### Web Speech API (word recognition)

- **Not exposed in Quest Browser** according to developer reports ([Meta forum, 2024](https://communityforums.atmeta.com/discussions/dev-quest/speechrecognition-in-webxr/1168273); [react-speech-recognition on Quest](https://communityforums.atmeta.com/discussions/dev-general/speech-to-text-library-react-speech-recognition-not-working-in-oculus-quest-2-br/1188891)). Nothing from Meta or the 2026 release notes says that changed. To be confirmed with one console check on Morten's headset.
- Even where it exists, Chromium's implementation has generally sent audio to a cloud service: a network dependency, latency of half a second or more, a privacy question, and language coverage issues for Danish and English players ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API/Using_the_Web_Speech_API)).
- Words also add little here. Citizens 5 mm tall reacting to "hello" is no better than them reacting to your voice.

### getUserMedia and WebAudio analysis

- `getUserMedia({ audio })` is a normal web API. Reports say that mic access granted outside the immersive session carries into it ([Babylon.js forum](https://forum.babylonjs.com/t/using-meta-quest-2-microphone-inside-of-webxr-game/38321)), and that prompts during an immersive session can interrupt it ([Babylon.js forum, camera permission](https://forum.babylonjs.com/t/webxr-camera-passthrough-support/58274)). Permission is per site and manageable in Quest Browser settings ([Meta help](https://www.meta.com/en-gb/help/quest/1406582186767703/)).
- Pitch: time-domain autocorrelation with parabolic interpolation is the robust, cheap choice for a single voice. YIN is the upgrade ([Alexander Ell: tuner with autocorrelation](https://alexanderell.is/posts/tuner/); [dev.to: autocorrelation vs FFT](https://dev.to/sendotltd/why-your-web-audio-tuner-sucks-at-low-notes-and-what-to-use-instead-of-fft-391h)). Latency is set by the analysis window: `fftSize` 2048 at 48 kHz is about 43 ms, well inside "feels instant".

### Recommendation

**Use getUserMedia plus WebAudio analysis. Do not use word recognition.**

- **Permission on the landing page, before entering:** a small optional switch, "Let them hear you". If it is on, `getUserMedia` is called from that click. The 2D prompt then never appears inside the headset session. If declined or unavailable, citizens respond to hands only, and nothing is lost.
- **Constraints:** `echoCancellation: true` (to subtract our own music and effects), `noiseSuppression: false` (it eats hums), `autoGainControl: false` (stable levels).
- **Analysis**, about 30 Hz on the main thread (an AudioWorklet if it costs too much): RMS level in dB against an **adaptive noise floor** (a slow-moving minimum), **onsets** from energy jumps, and **pitch** by autocorrelation only when the signal is voiced and loud enough.
- **Four reactions**, each obvious:
  - **Loud** (shout, cheer): they cheer back, jump, wave lanterns, with tiny synthesized "hooray" chirps.
  - **Sustained pitch** (hum, sing): they sway in time and **hum along a fifth above you**, and their lanterns brighten with your pitch.
  - **Quiet** (2 s of silence after sound): they hush, turn toward you and lean in.
  - **Sharp onset** (clap, knock on the desk): they startle and scatter, then creep back. Repeated claps make them clap along.
- **Our own sound:** duck the music while they listen, and ignore mic level that tracks our own output envelope. That is a backstop in case echo cancellation does not cover WebAudio output.

---

## 8. Technical architecture

### Reused from die Hände (copied and adapted, not imported)

| From die Hände | Into Petri Dish |
|---|---|
| Import map with three r150.1 from jsdelivr, `GLTFLoader`, `SkeletonUtils`, `BufferGeometryUtils` | Same |
| Renderer setup (alpha, ACES, sRGB, PCF shadows, `local-floor`) | Same |
| Session start and end, `begin(xr)` / `end()`, landing poster as live scene | Same, plus `hit-test`, `plane-detection` and `anchors` as optional features, and the mic switch |
| `PH[]` player hands: joints to lab space, openness, velocity, tracking loss, controller fallback | Same, plus pinch-point, pinch distance with hysteresis, hover |
| Depth-only hand occluders and hand shadows | Same |
| Room dimmer sphere with world-locked grain | Same, warmer, lighter, with a vignette term |
| Audio bus, HRTF panners, generated reverb, synthesized effects, soundtrack with a limiter | Same, plus the mic analysis |
| `placeLab(headPos, forward)` | Final fallback after plane detection and hit-test |
| `CFG`, `?sim`, `window.__T.go()`, shader pre-compile, canvas-texture labels after `document.fonts.load` | Same pattern |
| Studio footer, OG tags, canonical, `nxtwebxr.png`, README shape | Same |

### Files

```
nxt-petri-dish/
  index.html            the whole piece: landing page, lab, simulation, audio
  favicon.png           a new mark: a Petri dish seen from above (to make)
  nxtwebxr.png          studio mark, copied from die Hände
  assets/
    hand-left.glb       WebXR generic hands (MIT), copied
    hand-right.glb
    soundtrack.mp3      (to source, see open questions)
  docs/
    RESEARCH_AND_DIRECTION.md
  README.md
  DIRECTION.md          the short version of this file
  .nojekyll  .gitignore
```

Inside `index.html`, the same section banners die Hände uses: CONFIG, CORE, MATERIALS (glass, enamel, Bakelite, chrome, brass, walnut, rubber, environment map), LAB (tray, rack, droppers, bottles, dish stack, microscope, lamp, pedal bin), YOUR HANDS, PIPETTE, DROPS, DISH (sequence, flash, colony field), LIFE (sparks, cells, structures, beings), MICROSCOPE, VOICE, AUDIO, BEATS, PLACEMENT + LOOP, SESSION, SIM, BOOT.

### Libraries and versions

- three.js **0.150.1** (as die Hände, so its code moves over unchanged). Hit-test, plane detection and anchors are reached through the raw `XRFrame` and `XRSession` APIs, which do not need a newer three.js.
- Google Fonts: Jost, Oswald, Courier Prime, Yellowtail.
- Nothing else.

Upgrading three.js (r17x/r18x) would only be needed for built-in depth-sensing occlusion or WebGPU. Neither is in v1.

### Performance budget (Quest 3)

72 fps target. Framebuffer scale 1.3 and MSAA, as tested on the NXT feed. Fixed foveation low. About 100 draw calls. One shadow-casting light. Glass limited to the dish, lid, droppers and bottles. Instancing for everything alive. Simulation at 30 Hz, staggered across frames.

---

## 9. Game flow and state machine

```
POSTER  ──Enter (optional: "Let them hear you")──▶  PLACE
PLACE   find the desk: table plane → hit-test reticle → in front of you; pinch to set
        ──▶ UNPACK   the tray settles, the lamp clicks on, rack, bottles, dish stack, microscope
UNPACK  ──▶ OPEN     lift the dish lid (sticky pick), honey agar shines
OPEN    ──▶ LISTEN   the dish plays the sequence: flash + glyph + note
LISTEN  ──▶ DRIP     you answer with droppers
DRIP    ── wrong colour / second time-out ──▶ FAIL
        ── all correct ──▶ SUCCESS
FAIL    before life: DEAD (curdle, mould, smoke) ──▶ NEW DISH ──▶ LISTEN
        after life:  BLIGHT on a sector ──▶ GROW (cured by the next success)
SUCCESS ──▶ GROW     stage n+1 appears over 10 to 30 s
GROW    life evolves by itself; the dish asks for its next sequence when it is ready (it pulses)
        the microscope can be used at any time
        ── stage 6 reached ──▶ RECOGNITION
RECOGNITION  they build your hand in their architecture, they celebrate
        ──▶ GOD (sandbox, no ending, as die Hände): voice and hand reactions,
             more sequences grow the town, refill droppers, bring the scope back
```

Pacing target: first spark by about 2 minutes, citizens at 10 to 15 minutes.

---

## 10. Risks

1. **Pipette gestures on real hands.** Sticky pick is the safest design we know, but only the headset will tell. Prototype it first, with dwell-to-drip next to it.
2. **Microscope comfort.** Leaning into an eyepiece in MR may feel odd or clip with the near plane. The projection-screen fallback is ready.
3. **Readability of tiny life.** The 11-pixel budget is real. If stage 6 does not read, citizens get bigger and the dish gets bigger.
4. **Mic in the headset:** the permission flow, whether echo cancellation covers WebAudio output, and the Quest mic picking up desk knocks. Mitigated by asking on the landing page, ducking and the own-output gate. Needs a headset check early.
5. **Glass on Quest:** even the fake shader is transparent overdraw. Keep vessel count low and sort by hand.
6. **Placement:** without a room setup, plane detection returns nothing. Hit-test (Quest 3/3S) and the head-relative fallback cover it. Quest 2: grey passthrough and no depth hit-test, so it is a best-effort device.
7. **Tone:** "comical deadness" and the god theme can slide into parody. The deadpan narrator voice and the anti-slop list are the guard.
8. **Scope:** six stages, voice and a microscope is a lot. The order of work below keeps a playable game at every step.

Suggested order: placement and tray, droppers and drops (headset test), sequence and fail/success, stages 1 to 3 with the colony field, microscope, stages 4 to 6, voice, recognition and sandbox, sound and music, landing page.

---

## 11. Open questions for Morten

1. **Pipette gesture:** "sticky pick, pinch to squeeze" as the main control, with dwell-to-drip as an option? Or should picking up need a held pinch?
2. **Art bridge:** do you like the Bell Science Series idea (real lab, mid-century cartoon life) and the civilisation growing into 1950s Googie architecture with a monument of your hand?
3. **Stains:** green, red, yellow, then methylene blue as a fourth for longer sequences? And should colour blends in the agar shape the civilisation (builders, movers, lantern-makers)?
4. **Rhythm:** order only (recommended), or should timing count too?
5. **Failure after life exists:** a blight that the next success cures (recommended), or a full reset?
6. **Microscope:** eyepiece lean-in (recommended), or a projection screen?
7. **Voice:** fine to ask for the mic on the landing page as an optional switch? Any reactions to add or drop?
8. **Persistence:** should the civilisation be saved (localStorage plus a persistent anchor so the dish is still on your desk tomorrow), or is each visit a fresh dish, as die Hände has no save?
9. **Music:** a soundtrack like die Hände's? Space-age exotica, vibraphone and theremin fit. Do you have a source, or should it be synthesized?
10. **Hosting:** GitHub Pages mounted under nxtwebxr.com (as die Hände at `/diehaende/`), or the Cloudflare deploy the other mini-games use? What path, `/petridish/`?
11. **Desktop:** `?sim` developer preview only (as die Hände), or should non-headset visitors be able to play?
12. **Language:** English only, or German or Danish title cards like die Hände's bilingual cards?
