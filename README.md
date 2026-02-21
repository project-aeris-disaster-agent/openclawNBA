# Daredevil NBA Sports Data (OpenClaw Skill)

Request NBA schedule, scores, boxscore, standings, injuries, roster, and more. Payment per request via x402.

## What it does

- **Schedule** — Games for a date or date range, optional team filter.
- **Next game** — Upcoming game(s) for a team.
- **Boxscore / live score / play-by-play** — Game-level data (by game ID or team for today).
- **Standings** — NBA standings.
- **Injuries** — Current injury report.
- **Team roster** — Roster for a team.
- **Player stats** — Player statistics in context of a game.

## How to use

1. Install this skill into your OpenClaw workspace (e.g. via ClawHub: `clawhub install daredevil-nba-sportsdata` or copy the skill folder into your workspace `skills/`).
2. Ask your OpenClaw agent for NBA data, e.g.:
   - "What's the Lakers schedule this week?"
   - "When is the Celtics' next game?"
   - "NBA standings"
   - "Current injuries"
3. The agent will use the skill to call the Daredevil API and pay via x402, then return the data.

## Requirements

- OpenClaw workspace with network access.
- Ability to perform x402-paid HTTP requests (wallet on Base Sepolia testnet or Base mainnet, depending on seller config). The skill describes the endpoint and body; the agent needs an x402 client or tool to add payment and send the request.

## Base URL

The skill uses the public Daredevil seller:

**https://daredevil-acp-seller-production.up.railway.app**

- `GET /health` — Health check (no payment).
- `POST /v1/data` — Data request (x402 payment required).

## Request body (POST /v1/data)

JSON with at least `dataType`. Optional: `teamName`, `date`, `endDate`, `daysAhead`, `maxDaysAhead`, `nextGameLimit`, `gameId`. See `SKILL.md` for full schema.

## Troubleshooting

- **402 Payment Required** — Expected when no payment is sent. The agent must retry with x402 payment headers.
- **503** — x402 is unavailable on the server; try again later.
- **400** — Invalid body (e.g. missing or invalid `dataType`). Use one of: schedule, next_game, boxscore, live_score, play_by_play, standings, injuries, team_roster, player_stats.
