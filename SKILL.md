---
name: daredevil_nba_sportsdata
description: Request NBA sports data (schedule, boxscore, standings, injuries, roster, etc.) from the Daredevil seller. Payment per request via x402.
---

# Daredevil NBA Sports Data

When the user asks for NBA schedule, scores, boxscore, standings, injuries, team roster, or player stats, use this skill to request data from the Daredevil seller. Each request is paid via x402 (small fee per query).

## Base URL

**https://daredevil-acp-seller-production.up.railway.app**

- Health check (no payment): `GET /health`
- Data (payment required): `POST /v1/data` with JSON body and x402 payment headers

## Request (POST /v1/data)

Send a JSON body with:

| Field | Required | Description |
|-------|----------|-------------|
| `dataType` | Yes | One of: `schedule`, `next_game`, `boxscore`, `live_score`, `play_by_play`, `standings`, `injuries`, `team_roster`, `player_stats` |
| `teamName` | No | Team alias (e.g. LAL, BOS) or full name (Lakers, Celtics) |
| `date` | No | Date in YYYY-MM-DD format |
| `endDate` | No | End date for schedule range (YYYY-MM-DD) |
| `daysAhead` | No | For schedule: number of days ahead (e.g. 7 for one week) |
| `maxDaysAhead` | No | For next_game: max days to look ahead (default 14) |
| `nextGameLimit` | No | For next_game: max number of games to return (default 1) |
| `gameId` | No | Sportradar game ID when you need a specific game (boxscore, live_score, play_by_play) |

## Data types

- **schedule** — NBA games for a date or range (use `date`, `endDate`, or `daysAhead`; optional `teamName` to filter).
- **next_game** — Next upcoming game(s) for a team (`teamName` required; optional `nextGameLimit`, `maxDaysAhead`).
- **boxscore** — Full boxscore for a game (`gameId` or `teamName` for today’s game).
- **live_score** — Live score for a game (`gameId` or `teamName` for today’s game).
- **play_by_play** — Play-by-play for a game (`gameId` or `teamName` for today’s game).
- **standings** — NBA standings (optional `date`).
- **injuries** — Current injury report.
- **team_roster** — Team roster (`teamName` required).
- **player_stats** — Player stats (typically requires `gameId` or context from a game).

## Payment (x402)

The server requires payment for `POST /v1/data`. Use an x402-capable HTTP client:

1. First request without payment → server responds **402 Payment Required** with payment details in the `Payment-Required` header (or equivalent).
2. Create payment payload and sign with the user’s wallet (same network as server: Base Sepolia testnet `eip155:84532` or Base mainnet `eip155:8453`).
3. Retry `POST /v1/data` with the same body plus the x402 payment header(s). On success you get **200** and a JSON body with `dataType`, `result` (JSON string), and `timestamp`.

If your environment has a tool or script that performs x402-paid HTTP requests, use it with the base URL and request body above. Otherwise you may need to use `bash` with a CLI or script that supports x402 (e.g. Node script using `@x402/core` client).

## Response

Success (200) body shape:

```json
{
  "dataType": "schedule",
  "result": "<JSON string of the actual data>",
  "timestamp": "2025-02-21T...",
  "gameStatus": "optional for live games"
}
```

Parse `result` as JSON to get the underlying schedule, boxscore, standings, etc.

## Examples

- "What's the Lakers schedule this week?" → `dataType: "schedule", teamName: "LAL", daysAhead: 7`
- "When is the Celtics' next game?" → `dataType: "next_game", teamName: "BOS"`
- "NBA standings" → `dataType: "standings"`
- "Lakers roster" → `dataType: "team_roster", teamName: "LAL"`
- "Current NBA injuries" → `dataType: "injuries"`

Always use the base URL above and include x402 payment when calling `POST /v1/data`.
