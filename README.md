# Cricket Lineup Builder

Frontend-only (HTML, CSS, vanilla JS, JSON). No API keys, backend, or paid services.

## Run
- **Simplest:** open `index.html` in a browser (works from disk).
- **Or serve it** (reads the JSON files directly):
  `python3 -m http.server 8000` then visit http://localhost:8000
- Deploying? Upload the **whole folder** (index.html, frontend.html, css/, js/, data/, assets/) to the site root.

## Flow
`index.html` (pick International or IPL team) → **Continue** → `frontend.html` (lineup builder) → **← Change team** returns with the previous team selected. Lineups are not erased.

## Using the builder
- International = 11 positions (Playing XI). IPL = 12 positions (Playing XII) with separate **Batting first** / **Bowling first** lineups.
- Drag a player into a numbered position (mouse, or touch via the ⋮⋮ handle). Drag between positions to swap/move; drag back to the list to remove.
- Click alternative: tap a player, then tap a position. **+** adds to the next free position. Arrows/✕ reorder/remove.
- Duplicates are blocked; dropping on an occupied position replaces that player.
- Player lists: **Current squad**, **Potential & extended pool** (everyone, with their real team shown; filters: All, Other IPL Franchises, 2026 Auction Purchases, Unsold Auction Players, 2026 IPL Debutants, Uncapped, Historical IPL Players, Previous Auction Players, IPL Players, and Same nationality for international teams), **Recently active**.

## Data (what is and isn't verified)
Files: `data/teams.json` (teams, themes, squad ID lists), `data/players.json` (each player stored once, plus a `sources` table), `data/auctions.json`, and `data/data.js` (auto-generated fallback for `file://`).

**Last data update: 15 Nov 2026 (IPL 2026 auction + trades).** 336 player records.

- **IPL current squads** = the full 25-man IPL 2026 squads (official retained list of 15 Nov 2025 + the 77 players bought at the 16 Dec 2025 auction) with every trade confirmed up to 15 Nov 2026 applied. **IPL 2027 retentions are due 17 Nov 2026**, so these will change; update `teams.json` afterwards.
- **Auction (2026):** all 77 sales (team + price, accelerated-round flag) and all 79 unsold names are recorded on `player.auctions`. Capped/uncapped flags for the unsold come from Wisden; uncapped flags for the sold come from myKhel's uncapped list.
- **Statuses** (`player.statuses`, multiple allowed): `CURRENT_SQUAD` (recomputed by the site from `teams.json`), `AUCTION_BOUGHT`, `AUCTION_UNSOLD`, `IPL_DEBUTANT`, `HISTORICAL_IPL`, `UNCAPPED`. **Bought at auction is not the same as debuting.**
- **Debuts:** `IPL_DEBUTANT` is set only where a source reports an IPL match in 2026 (4 players). `debutStatus: "expected"` (11 players) means first-time IPL player per pre-season reports with the match not confirmed. Finn Allen shows why: in IPL squads since 2021, never played.
- **Historical IPL:** `HISTORICAL_IPL` is set for 41 players the maintainer knows played before 2026. **Per-season/franchise history from Cricsheet is NOT imported** (`iplMatchSeasons` is `null` for everyone). The "IPL Players (2016–2026)" filter therefore means IPL *squad membership* in the dataset.
- **International current squads: none loaded** (not verified). Those teams use the Extended Pool; "Same nationality" lists players in the dataset with that nationality.
- **Roles** are editorial (`roleSource: "editorial"`) except where a source describes the player (e.g. wicketkeeper-batter, pacer); `null` where unsure (111 players).
- **Unverified fields:** `unverifiedFields` lists assumptions. Nationality was assumed for the 76 unsold players (Wisden's table has no nationality) and for Zakary Foulkes (sources conflict).
- Auction/squad facts come from press reports of official events (IPL/BCCI advisories where noted), not from an official database. Every player has `source`, `sources`, `verification`, `lastVerified`; URLs are in `players.json -> sources`.

## Updating data (no JS changes needed)
1. Edit `data/teams.json` (a team's `squad` / `recentlyActive` are lists of player IDs) and `data/players.json` (add a player once, reference by ID). Use a trusted source (iplt20.com, official board announcements) and fill `source`, `verification`, `lastVerified`.
2. Run `python3 tools/make_data_js.py` to refresh the file:// fallback.
3. To add 2026 auction data again next year, copy `tools/add_2026_auction_data.py` as a template (it is idempotent: re-running it changes nothing).
4. For match history, download the free IPL JSON dataset from cricsheet.org, derive seasons per player, and write them into `iplMatchSeasons`. Respect each source's terms; do not scrape.

## Storage keys (localStorage)
`cricket_selected_team`, `cricket_lineups_v2` (keys like `srh:bat`, `srh:bowl`, `ind:xi`), `cricket_prefs_v1`.

## Structure
```
index.html  frontend.html  css/styles.css
js/{storage,player-database,team-selection,drag-drop,app}.js
data/{teams,players,auctions}.json  data/data.js  tools/{make_data_js,add_2026_auction_data}.py  assets/logos/
```
