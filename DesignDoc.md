# SHADOW GOVERNMENT — Reboot GDD v1 (DesignDoc.md)

_A satirical strategy card game where conspiracy theories become policy._

---

## 1) Vision & Pillars
- **Tone/Theme:** Satirical conspiracy‑humor with X‑Files / Weekly World News vibes framed as a tongue‑in‑cheek newspaper.
- **Core Loop:** Draw → play up to **3** cards (pay IP) → resolve effects (states/opponent/map) → **25%** random event → End Turn → AI mirrors the same rules.
- **Win / Loss:** Control **10** states **or** reach **200 IP** **or** Truth ≥ **90%** (Truth Seekers) / ≤ **10%** (Government) **or** complete **Secret Agenda**.
- **Identity:** Periodic **glitch** renames the newspaper/page title and skews UI slightly for one screen.
- **Modularity:** Standalone HTML has a **light built‑in rules/cards** fallback. If present, it auto‑loads external modules (JS) and **expansion packs**.

---

## 2) Platform & Tech
- **Stack:** Single‑file **HTML/CSS/Vanilla JS** app; no frameworks.
- **Graphics:** SVG + **D3** + **TopoJSON** for the USA map (grid fallback if offline).
- **Persistence:** `localStorage` for options, expansions, autosave.
- **Audio:** HTML5 `<audio>` with grouped playlists (Theme/Government/Truth), loop/shuffle, crossfade.

---

## 3) Project Layout (file structure)
```
/ (root)
  index.html                  # the playable app; embeds fallbacks
  /js
    Rules.js                  # core rules & constants (VC, timing, effects)
    CoreCards.js              # complete base card set (both factions)
    Map.js                    # topo loader, render, coloring, tooltips
    Newspaper.js              # “The Paranoid Times” overlay & content builder
    AudioController.js        # music/SFX mixer, crossfade, now‑playing
    OpponentAI.js             # simple, transparent AI that follows same rules
    Expansions.js             # registry, toggle UI, deck build
  /expansions                 # drop‑in, independent packs (pure JS)
    floridaman.js
    moontruth.js
  /muzak                      # audio files (mp3)
    Theme-1.mp3, Theme-2.mp3
    Government-1.mp3, ...
    Truth-1.mp3, ...
  /map
    us.json                   # TopoJSON (states‑10m or simplified)
```
> **Rule:** Only these 3 subfolders: **`js/`**, **`expansions/`**, **`muzak/`**. (Map may be embedded or fetched; use grid fallback if missing.)

---

## 4) Start Screen & Global UI
### 4.1 Start Menu
- **New Game**
- **Manage Expansions** (modal with toggles; persist selection)
- **How to Play** (modal)
- **Continue** (autosave)
- **Credits**
- **Options** (identical both places; see below)

### 4.2 Options (same UI on start & in‑game)
- **UI Scale:** Small / Medium / Large (CSS variables).
- **High Contrast:** on/off; **Text Size:** S / M / L.
- **Audio:** Master/Music/SFX sliders, **Shuffle/Loop** toggles, **Crossfade (ms)**, **Now Playing** label.
- **Toggles:** AI Panel, Deck Tracker, Particle FX, Screen Shake.
- **Animation Speed:** Slow / Normal / Fast / Instant.
- **Difficulty:** Easy / Normal / Hard / **Conspiracy** (more AI actions per turn).
- **Main Menu** (in‑game only), **View Stats** (W/L, streak).

### 4.3 Faction Select
- **Government (Deep State):** +10 IP start.
- **Truth Seekers:** +10% starting Truth (Truth = 60% if human is Truth).
- Music switches to **Government-\*** or **Truth-\*** upon selection.

---

## 5) In‑Game Screen (newspaper dashboard)
- **Header:** Masthead **THE PARANOID TIMES** (glitch may rename), subtitling, ticker.
- **Faction Banner:** “Government Operative” / “Truth Seeker Operative”.
- **Stats Bar:** Your IP, Truth %, States, Round/Phase/Combo, AI IP.
- **Left Pane:** Victory Conditions, Classified Intel (agenda text), Log.
- **Center:** **USA Map** (see §7) with ownership colors & pressure badges.
- **Right Pane:** AI Panel (hand size facedown, strategy), Your Hand (minimized cards).
- **Bottom Bar:** **Play Card**, **End Turn [Space]**, **Upgrade [U]**, **Abandon**.
- **Deck Tracker:** cards played, tiny quips.

---

## 6) Rules (authoritative)
### 6.1 Start & Resources
- **Truth baseline:** 50%.
- If **player = Government** → +10 IP.
- If **player = Truth** → Truth starts at **60%**.
- **Base Income:** +5 IP per turn.
- **IP floor:** never below 0; **Truth clamp:** [0,100].

### 6.2 States & Defense
- Defense tiers: **2** (most), **3** (CA/NY/TX/FL/PA/IL, etc.), **4** (DC, AK, HI).
- **Income per state:** 2 / 3 / 4 IP for Defense 2 / 3 / 4.
- **Pressure** (per side per state): added by Zone/Tech; when **Pressure ≥ Defense**, you **capture** in **Resolution**; then both pressures reset to 0.
- **Special States:**
  - **California:** Tech cards **−2 IP**.
  - **New York:** Media cards **−2 IP**.
  - **Texas:** +2 IP per turn.
  - **Florida:** Reaction tie‑break (Defensive vs Defensive) favors **owner**.
  - **Washington D.C.:** +5 to **your** Truth manipulation cards.

### 6.3 Turn Structure (per player)
1) **Income Phase:** +5 + state income + developments. Draw **1** (hand max **7**).
2) **Action Phase:** Play up to **3** cards (pay IP; choose targets).
   - **Reaction Window:** When a harmful card targets you/your state → defender may play **one** **Defensive/Instant**; then active player may play **one** **Instant**; resolve **LIFO**; stop chain.
3) **Resolution Phase:** Apply effects; check captures; reset pressures.
4) **Event Phase:** **25%** chance: draw a **Random Event** (see §10).
5) **Victory Check:** If any condition is met, end immediately.
6) **Switch Turn** (AI follows same rules).

### 6.4 Victory Conditions
- **10** states **OR** **200 IP** **OR** Truth **≥90%** (Truth) / **≤10%** (Gov) **OR** **Secret Agenda** completed.
- Truth thresholds end the game **immediately** when crossed on the acting player’s turn.

---

## 7) USA Map (Map.js)
- **Data:** `/map/us.json` (TopoJSON). If unavailable → **grid fallback** (10×5 cells).
- **Projection:** `d3.geoAlbersUsa()`.
- **Interactions:** hover glow + tooltip (name/owner/defense/pressure), click to choose **target** state.
- **Colors:** Player (blue), AI (red), Neutral (gray), **Contested** (pulsing orange).
- **Badges:** Small numeric chips near centroids for **your/AI pressure** when > 0.
- **Performance:** Diff‑coloring/badges only on change.

---

## 8) Card System
### 8.1 Schema (shared for core & expansions)
```ts
type Rarity = 'common'|'uncommon'|'rare'|'legendary';

type Effect =
 | { k:'ip', who:'player'|'ai', v:number }
 | { k:'truth', who:'player'|'ai', v:number }
 | { k:'pressure', who:'player'|'ai', state:string, v:number }
 | { k:'defense', state:string, v:1|-1 }
 | { k:'draw', who:'player'|'ai', n:number }
 | { k:'discardRandom', who:'player'|'ai', n:number }
 | { k:'addCard', who:'player'|'ai', cardId:string }
 | { k:'flag', name:string, on?:boolean }
 | { k:'special', fn:(gs,target)=>void };

type Card = {
  id: string;
  name: string;
  type: 'MEDIA'|'ZONE'|'ATTACK'|'TECH'|'DEVELOPMENT'|'DEFENSIVE'|'INSTANT';
  cost: number;
  rarity?: Rarity;
  text: string;
  flavor?: string;
  target?: 'state'|'player'|'ai'|'global'|null;
  effects: Effect[] | ((ctx:{gs,target})=>Effect[]);
  aiWeight?: number;
};
```

### 8.2 Base Card Examples
- **Leaked Docs** — *ATTACK*, Cost 5 → Opponent **−8 IP**.
- **Disinformation** — *MEDIA*, Cost 4 → **Truth −10** against opponent side.
- **Black Budget** — *DEVELOPMENT*, Cost 3 → **+10 IP next round** (one‑shot).
- **Bunker** — *DEFENSIVE*, Cost 4 → **Immune** to attacks this round.
- **Redacted Document** — *DEFENSIVE*, Cost 4 → **Counter** next **Attack/Media** targeting you/your state (only 1 card).
- **Media Smear** — *ATTACK*, Cost 6 → Opponent **gains 0 IP next round**.
- **Moon Landing Hoax** — *MEDIA*, Cost 8 → **Truth +15** for you (**+5** extra if you own **D.C.**).
- **Local Influence** — *ZONE*, Cost 6 → **+1 Pressure** in chosen state.
- **Crisis Actors** — *INSTANT/MEDIA*, Cost 7 → Opponent **skips next action** this turn.
- **Think Tank** — *DEVELOPMENT*, Cost 10 → **+1 IP/turn**, max **3** active developments.

- **Hand:** Start **5** cards; **hand limit 7**; reshuffle discard into draw when empty.

---

## 9) Secret Agendas
Examples (random at game start; purple panel shows name + progress bar):
- **Perfect Control:** Control **15** states.
- **Financial Domination:** Reach **250 IP**.
- **The Great Awakening:** Reach **100 Truth** and keep it for **2** consecutive rounds.
- **Palace Coup:** Control **D.C.** + **2** adjacent states.
`progressFn(gs)` returns a 0–1 value; completed agendas trigger instant victory.

---

## 10) Random Events (25% after each **player** turn)
- **Solar Storm:** All states **Defense −1** (min 1) this round.
- **Economic Boom:** Both **+10 IP**.
- **Cyber Attack:** Both **discard 1** at random.
- **Peace Protest:** **Truth Seekers +5 Truth**.
- **Security Crackdown:** **Government** gets **+1 Defense** in owned states this round.
- **Pandemic:** Both **−1 draw next round** (min 0).
- **Industrial Accident:** Random state **clears all Pressure**.
- **UN Sanctions:** Both **−5 IP** (min 0).
- **Nuclear Scare:** **Truth −5** (global).
- **Spy Scandal:** Side **leading in Truth** loses **5 IP**.

Events create log entries and satirical blurbs in the newspaper overlay.

---

## 11) AI (OpponentAI.js)
- **Fair Play:** Same hand limits, IP costs, reaction window (max 1 defensive/instant per trigger).
- **Priorities:** (1) **Win now**, (2) **Deny/counter**, (3) **Economy**, (4) **Expand**.
- **Difficulty:** controls number of plays it attempts per turn (e.g., 1/2/3/4).
- **Target choice:** evaluate states by (defense − current pressure) and state bonuses.

---

## 12) Newspaper (Newspaper.js)
- **Trigger:** Show at Round 1; then every round (or at least regular cadence).
- **Layout:** Continue ribbon, price ($4.20), masthead, black headline band, two 2‑column rows, ad box, bottom ticker.
- **Sources:** Recent GameLog (captures, truth swings, big plays) + random satire templates + ad bank.
- **Glitch:** 2–5% chance to swap masthead to **THE PARANOIA TIMES**, **THE BIRDS ARE DRONES**, **WAKE UP SHEEPLE**, etc., and rotate page title + subtle CSS skew for one screen.

---

## 13) Expansions (Expansions.js)
- **Format (pure JS):**
```js
registerExpansion({
  name: "Florida Man",
  version: "1.0.0",
  icon: "🦩",
  cards: [ /* Card[] */ ],
  onAttach(gs){},
  onDetach(gs){}
});
```
- **Registry:** `EXP_REGISTRY[]`; toggles in **Manage Expansions**; persisted in `localStorage.sg_selected_packs`.
- **Deck Build:** **Core** + enabled packs at **New Game**.
- **Custom Import (later):** allow JSON via file picker; validate against schema.

---

## 14) Audio (AudioController.js)
- **Groups:** `Theme-*`, `Government-*`, `Truth-*`.
- **Behavior:** Start screen = Theme; in game = faction group.
- **Controls:** music/SFX enable, now‑playing label, shuffle/loop, **crossfadeMs**.
- **SFX:** “deal”, “capture”, “event”, “click”.

---

## 15) Save/Load
- **Autosave** at **End Turn** → `localStorage.sg_save`.
- **Continue** from start screen; **Quick Save/Load** (Q/L) in options.
- Save state includes: deck/discard/hands, IP/truth, pressures, owners, developments, options.

---

## 16) Accessibility & Responsiveness
- **UI Scale** via CSS vars; **Text Size** & **High Contrast** classes.
- **Keyboard:** Space (End Turn), **1–7** (select card), **U** (Upgrades), **S** (Stats), **Q/L** (Quick Save/Load).
- **Mobile:** Large touch targets; panels avoid covering the **map**.

---

## 17) Module APIs (contracts)
```ts
// Core App
App.init()
App.newGame(opts)                    // applies expansions, faction bonuses
App.load(save)

// Rules
Rules.VICTORY_CONDITIONS
Rules.STATE_DEFENSE
Rules.applyEffects(gs, effects:Effect[])
Rules.checkVictory(gs)

// Map
Map.init(el, topo)
Map.setOwner(stateId, owner:'player'|'ai'|null)
Map.setPressure(stateId, pPlayer:number, pAI:number)
Map.highlight(stateId)
Map.update()                         // diff‑coloring, badges

// Newspaper
News.showTurn(gs, highlights: string[])
News.breaking(title, text)

// Audio
Audio.playMusic(group, {shuffle, loop, crossfadeMs})
Audio.playSFX(name)

// Expansions
registerExpansion(pack)
Exp.setEnabled(name, bool)
Exp.buildDeck(coreCards)             // returns Card[]
```

---

## 18) Built‑in Fallback (standalone)
If external JS fails to load, the **HTML** provides:
- **Light Rule Set:** victory thresholds, defense map, event table.
- **Light Card Set:** ~10 core cards (both sides) with **real effects**.
- **Newspaper** & **Glitch** baked‑in.
This ensures the game **runs** even without `/js/*` present.

---

## 19) QA Checklist
- Start screen buttons work on desktop + Android (Desktop‑site).
- **Options** identical on start & in‑game; **Main Menu** returns safely.
- **How to Play** is a **modal** with clear rules.
- **Expansions** modal lists packs; toggles persist; deck reflects toggles.
- **Music**: Theme → Government/Truth after faction pick; Now Playing updates; SFX fire.
- **Map**: TopoJSON loads; fallback grid when missing.
- **Pressure → Capture** works; reset on capture; income/bonuses next Income.
- **Reaction window** enforces 1‑vs‑1 **LIFO** chain.
- **Random Events** roll 25% after each player turn and feed the newspaper.
- **Truth clamp** [0,100]; threshold wins end **immediately**.
- **Autosave** on End Turn; **Continue** restores including pressures & developments.
- **Glitch** occasionally renames masthead/title and resets next screen.

---

## 20) Milestones
1. **Rules parity pass** (thresholds, bonuses, reaction window).
2. **Card pool**: grow to ~40 core cards with rarity/weights.
3. **AI heuristics**: scoring per state & card; “threat of capture” detector.
4. **Newspaper writers**: more templates + per‑event blurbs; rotating ad bank.
5. **Expansions SDK**: doc + example pack (“Florida Man”).
6. **A11y polish**: focus order, ARIA labels, contrast audit.

---

**Owner:** Tom (Shadow Government)  
**Version:** 1.0 — _DesignDoc.md generated from the reboot brief._
