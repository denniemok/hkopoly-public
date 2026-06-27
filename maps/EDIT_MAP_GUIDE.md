# Map Editing Guide

繁體中文：[EDIT_MAP_GUIDE.zh-HK.md](EDIT_MAP_GUIDE.zh-HK.md)

Welcome! This guide explains the JSON structure for HKOPOLY board maps and how to create or edit them.

## Locales

Game content is split by locale. Each locale has its own folder under `maps/`:

| Locale | Folder | Live site |
|--------|--------|-----------|
| English | `maps/en/` | [hkopoly.com](https://hkopoly.com) |
| Traditional Chinese (HK) | `maps/zh-HK/` | [zh.hkopoly.com](https://zh.hkopoly.com) |

The English and Chinese sites run on **separate servers**—rooms and players are not shared.

Board **layout** (`id` 0–39, prices, rents, groups) is the same across locales; **display names** (`name`, `subtitle`, `groups[].name`, etc.) are localized per folder. When contributing a new map, please provide JSON for **both** locales unless the theme is intentionally single-locale.

Reference files: `maps/en/urban1.json`, `maps/zh-HK/urban1.json`.

---

## File structure

Each map JSON has five top-level fields:

| Field | Description |
|-------|-------------|
| `id` | Unique map id (e.g. `urban1`) — same across locales |
| `name` | Display name (localized, e.g. `Urban 1` / `市區 1`) |
| `config` | Game rule settings (shared values; see below) |
| `board` | 40-space board definition (`id` 0–39) |
| `groups` | Property colour group definitions |

---

## Board layout (`id` 0–39)

The board has **40 spaces**. Players move clockwise: `0 → 1 → 2 → … → 39 → 0`.

```
  0 ─── 1 ─── 2 ─── 3 ─── 4 ─── 5 ─── 6 ─── 7 ─── 8 ─── 9 ── 10
  │                                                           │
 39                                                          11
  │                                                           │
 38                                                          12
  │                                                           │
 37                                                          13
  │                                                           │
 36                                                          14
  │                                                           │
 35                        (board centre)                    15
  │                                                           │
 34                                                          16
  │                                                           │
 33                                                          17
  │                                                           │
 32                                                          18
  │                                                           │
 31                                                          19
  │                                                           │
 30 ── 29 ── 28 ── 27 ── 26 ── 25 ── 24 ── 23 ── 22 ── 21 ── 20
```

| Position | `id` | Corner / edge | Default use |
|----------|------|---------------|-------------|
| Top-left | **0** | Top row start | `go` |
| Top row (L→R) | **1–9** | Top edge | Properties, Chance, tax, etc. |
| Top-right | **10** | Top row end | `jail` (visiting only) |
| Right (T→B) | **11–19** | Right edge | Properties, Community Chest, etc. |
| Bottom-right | **20** | Bottom row end | `parking` (Free Parking) |
| Bottom row (R→L) | **21–29** | Bottom edge | Properties, Chance, etc. |
| Bottom-left | **30** | Bottom row start | `gotojail` |
| Left (B→T) | **31–39** | Left edge | Properties, tax, etc. |

> **Note:** `id` 0 is **top-left** on screen (Go), not the classic Monopoly bottom-right. `id` 10 must be `jail`; `id` 30 must be `gotojail`.

---

## Space types (`board[].type`)

| `type` | Description | Common fields |
|--------|-------------|---------------|
| `go` | Go | `name`, `subtitle` (optional pass/land bonus text) |
| `property` | Property | `name`, `price`, `rent`, `group` |
| `railroad` | Railway / MTR | `name`, `price`, `rent`, `group: "railroad"` |
| `utility` | Utility | `name`, `price`, `group: "utility"` |
| `chance` | Chance | `name` |
| `community` | Community Chest | `name` |
| `tax` | Tax | `name`, `amount` |
| `jail` | Jail (visiting only) | `name`, `cellLabel` |
| `parking` | Free Parking | `name` |
| `gotojail` | Go to Jail | `name` |

### Rent array `rent`

Property `rent` is a length-6 array:

`[unimproved, 1 house, 2 houses, 3 houses, 4 houses, hotel]`

Railroad `rent` is length 4, scaling with 1–4 lines owned.

---

## Property groups (`groups`)

Each colour group needs:

```json
"shamShuiPo": {
  "id": "shamShuiPo",
  "name": "Sham Shui Po",
  "color": "#04642c",
  "icon": "/icons/hk_18d_ssp.webp",
  "spaceIds": [6, 8, 9],
  "houseCost": 50
}
```

| Field | Description |
|-------|-------------|
| `id` | Group key; must match `board[].group` |
| `name` | Display name (localized) |
| `color` | Colour band (HEX) |
| `icon` | Icon path or Lucide name (see below) |
| `spaceIds` | Array of space `id`s in this group |
| `houseCost` | House/hotel build cost (omit for railroad / utility) |

Each map should have **8 property colour groups**, **1 railroad group** (`railroad`), and **1 utility group** (`utility`).

---

## Icons (`groups[].icon`)

### Hong Kong 18-district badges (WebP)

Property groups may use built-in district icons at `/icons/hk_18d_<code>.webp`:

| Code | Path | District |
|------|------|----------|
| `central` | `/icons/hk_18d_central.webp` | Central & Western |
| `wc` | `/icons/hk_18d_wc.webp` | Wan Chai |
| `east` | `/icons/hk_18d_east.webp` | Eastern |
| `south` | `/icons/hk_18d_south.webp` | Southern |
| `ytm` | `/icons/hk_18d_ytm.webp` | Yau Tsim Mong |
| `ssp` | `/icons/hk_18d_ssp.webp` | Sham Shui Po |
| `kc` | `/icons/hk_18d_kc.webp` | Kowloon City |
| `kt` | `/icons/hk_18d_kt.webp` | Kwun Tong |
| `wts` | `/icons/hk_18d_wts.webp` | Wong Tai Sin |
| `north` | `/icons/hk_18d_north.webp` | North |
| `tp` | `/icons/hk_18d_tp.webp` | Tai Po |
| `st` | `/icons/hk_18d_st.webp` | Sha Tin |
| `sk` | `/icons/hk_18d_sk.webp` | Sai Kung |
| `tm` | `/icons/hk_18d_tm.webp` | Tuen Mun |
| `yl` | `/icons/hk_18d_yl.webp` | Yuen Long |
| `tw` | `/icons/hk_18d_tw.webp` | Tsuen Wan |
| `kwt` | `/icons/hk_18d_kwt.webp` | Kwai Tsing |
| `island` | `/icons/hk_18d_island.webp` | Islands |

### Lucide icons (kebab-case)

If `icon` does **not** start with `/`, it is treated as a [Lucide](https://lucide.dev/icons/) icon name (kebab-case).

| Group | `icon` value |
|-------|--------------|
| Railroad | `tram-front` |
| Utility | `zap` |

Other options: `train-front`, `building-2`, `landmark`, `droplets`, `flame`, etc.

---

## Game config (`config`)

```json
{
  "GO_SALARY": 200,
  "GO_LANDING_BONUS": 100,
  "JAIL_POSITION": 10,
  "JAIL_BAIL": 50,
  "OWNABLE_SPACE_TYPES": ["property", "railroad", "utility"],
  "PLAYER_COLORS": ["#C0DA5A", "#FFC13F", "..."]
}
```

| Field | Description |
|-------|-------------|
| `GO_SALARY` | Salary for passing Go |
| `GO_LANDING_BONUS` | Extra bonus for landing on Go (pass + land = total) |
| `JAIL_POSITION` | Jail space `id` (always 10) |
| `JAIL_BAIL` | Bail amount |
| `OWNABLE_SPACE_TYPES` | Purchasable space types |
| `PLAYER_COLORS` | Player token colours (up to 12; use the same palette as existing maps) |

> Keep `config` values aligned with existing maps unless you are deliberately proposing a rule change.

---

## Checklist

- [ ] JSON placed under `maps/en/` and/or `maps/zh-HK/` as appropriate
- [ ] `board` length is **40**; `id` **0–39** unique and consecutive
- [ ] `id` 0 = `go`, 10 = `jail`, 20 = `parking`, 30 = `gotojail`
- [ ] Every `property` / `railroad` / `utility` `group` exists in `groups`
- [ ] Each group's `spaceIds` matches actual space `id`s
- [ ] Railroad group has 4 spaces; utility group has 2
- [ ] ~3 Chance and ~3 Community Chest spaces (matches existing maps)
- [ ] 2 tax spaces

---

## Submitting a new map

1. Add `<your-map-id>.json` under `maps/en/` and `maps/zh-HK/` (or the locale(s) you support)
2. Use `urban1.json` / `rural1.json` in each locale folder as references
3. Open a Pull Request with a short description of the theme and districts used

We welcome new Hong Kong–themed boards and other localized themes!
