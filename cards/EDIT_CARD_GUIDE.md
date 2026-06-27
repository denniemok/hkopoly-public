# Card Editing Guide

繁體中文：[EDIT_CARD_GUIDE.zh-HK.md](EDIT_CARD_GUIDE.zh-HK.md)

Welcome! This guide explains the JSON structure for HKOPOLY Chance and Community Chest cards.

## Locales

Card text is localized per locale. Each locale has its own file:

| Locale | Path | Live site |
|--------|------|-----------|
| English | `cards/en/cards.json` | [hkopoly.com](https://hkopoly.com) |
| Traditional Chinese (HK) | `cards/zh-HK/cards.json` | [zh.hkopoly.com](https://zh.hkopoly.com) |

The English and Chinese sites run on **separate servers**—rooms and players are not shared.

Card **logic** (`action`, `target`, `amount`, etc.) should stay the same across locales; only **`text`** (and currency wording in descriptions) is translated. When editing cards, update **both** locale files unless the change is English- or Chinese-only by design.

---

## File structure

`cards.json` has three top-level fields:

| Field | Description |
|-------|-------------|
| `spacePlaceholder` | Placeholder for move-card destinations (always `{space}`) |
| `chance` | Chance card deck (array) |
| `community` | Community Chest deck (array) |

Each card needs at least `text` (display string) and `action` (game logic). Other fields depend on the `action` type.

---

## Basic card format

```json
{
  "text": "Bank dividend HK$50.",
  "action": "money",
  "amount": 50
}
```

| Field | Description |
|-------|-------------|
| `text` | Description shown when drawn (supports `{space}` placeholder; see below) |
| `action` | Game logic triggered by the card |

---

## Placeholder `{space}`

`spacePlaceholder` defaults to `{space}`. For cards with `action: "move"` and a `target`, the game replaces `{space}` with the target board space's `name` from the **active map**.

```json
{
  "text": "Advance to {space}. If you pass Go, collect HK$200.",
  "action": "move",
  "target": 13
}
```

If space `id` 13 is named "Mong Kok", the player sees: "Advance to Mong Kok. If you pass Go, collect HK$200."

> **Note:** `target` must be a valid space `id` (0–39) on the board. Space **names** differ by locale/map theme, but **positions** (`id`) are consistent across maps.

---

## Action types (`action`)

### `move` — Move to a space

Moves the player directly to `target` and triggers that space's effects (buy, pay rent, etc.).

| Field | Required | Description |
|-------|----------|-------------|
| `target` | Yes | Target space `id` (0–39) |
| `collectGo` | No | Whether passing Go pays salary. Default `true`; set `false` to skip |

Passing Go uses `config.GO_SALARY` (default HK$200). Landing on Go (`id` 0) also adds `config.GO_LANDING_BONUS` (default HK$100).

```json
{ "text": "Advance to {space}. Collect HK$300.", "action": "move", "target": 0, "collectGo": true }
```

---

### `money` — Receive or pay cash

| Field | Required | Description |
|-------|----------|-------------|
| `amount` | Yes | Positive = income; negative = payment |

If the player cannot pay, mortgage / sell flows apply; bankruptcy may follow.

```json
{ "text": "Bank dividend HK$50.", "action": "money", "amount": 50 }
{ "text": "Pay poor tax HK$15.", "action": "money", "amount": -15 }
```

---

### `jailCard` — Get Out of Jail Free

Player receives a Get Out of Jail Free card (usable in jail or tradable).

```json
{ "text": "Get Out of Jail Free card.", "action": "jailCard" }
```

---

### `jail` — Go to Jail

Player goes to jail immediately—**does not** pass Go or collect salary. Turn ends.

```json
{ "text": "Go to Jail. Do not pass Go.", "action": "jail" }
```

---

### `back` — Move backward

Player moves back `spaces` and triggers landing effects.

| Field | Required | Description |
|-------|----------|-------------|
| `spaces` | Yes | Number of spaces to move back (positive integer) |

```json
{ "text": "Go back 3 spaces.", "action": "back", "spaces": 3 }
```

---

### `nearestRailroad` — Advance to nearest railroad

Moves the player to the **next** railroad space (`group: "railroad"`). If none ahead, wraps to the first railroad on the board.

| Field | Required | Description |
|-------|----------|-------------|
| `payDouble` | No | If `true`, rent is doubled when the railroad is owned |

```json
{ "text": "Advance to the nearest MTR line. Pay double rent.", "action": "nearestRailroad", "payDouble": true }
{ "text": "Advance to the nearest MTR line.", "action": "nearestRailroad" }
```

---

### `repairs` — Building repairs

Charges based on houses/hotels on owned properties.

| Field | Required | Description |
|-------|----------|-------------|
| `house` | Yes | Cost per house (1–4 houses) |
| `hotel` | Yes | Cost per hotel (5 houses) |

Only **owned** properties count; vacant and mortgaged properties are excluded.

```json
{
  "text": "General repairs: HK$25 per house, HK$100 per hotel.",
  "action": "repairs",
  "house": 25,
  "hotel": 100
}
```

---

### `payEach` — Pay each player

Drawing player pays `amount` to **every** other active player. Insufficient funds may trigger mortgage flows per recipient.

```json
{ "text": "You are elected chairman. Pay each player HK$50.", "action": "payEach", "amount": 50 }
```

---

### `collectEach` — Collect from each player

Every other active player pays `amount` to the drawing player.

```json
{ "text": "It is your birthday. Collect HK$10 from each player.", "action": "collectEach", "amount": 10 }
```

---

## Two decks

| Deck | Field | Triggered by |
|------|-------|--------------|
| Chance | `chance` | Landing on `type: "chance"` |
| Community Chest | `community` | Landing on `type: "community"` |

Decks are **shuffled independently** with separate discard piles; empty decks are reshuffled.

Aim for **16 cards** per deck (matches the current set); count is flexible.

---

## Relationship to maps

Cards and maps are stored separately:

- Card text and logic → `cards/<locale>/cards.json`
- Board spaces and rules → `maps/<locale>/<map-id>.json`

These **map** `config` values affect card behaviour (cards do not override them):

| Setting | Affects |
|---------|---------|
| `GO_SALARY` | `move` passing Go bonus |
| `GO_LANDING_BONUS` | Extra bonus for landing on Go (`id` 0) |
| `JAIL_POSITION` | `jail` destination |
| Railroad positions | `nearestRailroad` targets |

When setting `move` card `target` values, see the board layout in [`maps/EDIT_MAP_GUIDE.md`](../maps/EDIT_MAP_GUIDE.md).

---

## Checklist

- [ ] Updated `cards/en/cards.json` and/or `cards/zh-HK/cards.json` as needed
- [ ] Every card has `text` and `action`
- [ ] `action` is a supported type (see above)
- [ ] `move` `target` is 0–39 and points to a sensible space
- [ ] `money` / `payEach` / `collectEach` include `amount`
- [ ] `back` includes `spaces`
- [ ] `repairs` includes both `house` and `hotel`
- [ ] Text amounts match `amount` / rate fields
- [ ] Same card logic across locales (only `text` differs)
- [ ] Valid JSON

---

## Submitting changes

1. Edit `cards/en/cards.json` and `cards/zh-HK/cards.json` (or the locale(s) you support)
2. Confirm fields and action types against this guide
3. Open a Pull Request with a short summary (localization, balance, new cards, etc.)

We welcome flavourful Chance and Community Chest copy!
