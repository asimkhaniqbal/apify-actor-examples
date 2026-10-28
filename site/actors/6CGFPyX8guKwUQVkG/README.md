# SofaScore Live Scores, Standings & Match Data

Scrape **SofaScore** live scores, fixtures, league tables, team and player profiles, match statistics, lineups, odds and rankings across 25 sports. Search by name, paste SofaScore URLs, or pull every live and scheduled match. No login, API key or proxy setup.

## What this actor does

- **Nine ways in:** search by name, live events, fixtures by date, team profile, player profile, tournament profile, standings, match detail by ID/URL, and rankings.
- **25 sports:** football, basketball, tennis, ice hockey, cricket, MMA, esports, volleyball, handball, rugby, and more.
- **Match depth on demand:** statistics, lineups with ratings, incidents, odds, crowd votes, head-to-head, shotmap, momentum, average positions, best players, win probability, managers, streaks and commentary.
- **Team and player data:** squad lists, manager, venue, market value, contract, nationality and per-competition season statistics.
- **Leagues and rankings:** overall, home and away tables, group tables, FIFA, UEFA, ATP, WTA, UTR, FIP, rugby, UFC and FNC rankings.
- **Empty fields are omitted** — a field SofaScore does not publish for a record is left out, never filled with `null` or a placeholder.

## Output per record

Every row has a `recordType`, a `sofascoreUrl` where one exists, and `scrapedAt`.

### `recordType: "event"`

- `eventId`, `slug`, `sport`, `startTime`, `startTimestamp`, `status`, `statusCode`, `statusDescription`, `winnerCode`
- `homeTeam` / `awayTeam` — id, name, slug, shortName, nameCode, logoUrl, teamColors, countryCode, manager
- `homeScore` / `awayScore`, `currentPeriod`, `matchTime`
- `tournamentName`, `tournamentId`, `uniqueTournamentId`, `uniqueTournamentName`, `categoryName`, `country`, `countryCode`
- `seasonName`, `seasonId`, `seasonYear`, `round`, `roundName`
- `venue`, `city`, `venueCapacity`, `venueCoordinates`, `referee`, `refereeCountry`, `attendance`
- `hasStatistics`, `hasXg`, `hasHighlights`
- Optional: `statistics`, `lineups`, `incidents`, `odds`, `votes`, `h2h`, `advanced` (shotmap, momentum, average positions, best players, win probability, managers, team streaks, pre-game form), `comments`

### `recordType: "team"`

- `teamId`, `name`, `slug`, `shortName`, `fullName`, `nameCode`, `sport`
- `country`, `countryCode`, `category`, `tournamentId`, `tournamentName`, `uniqueTournamentId`
- `manager`, `managerCountry`, `venue`, `venueCapacity`, `venueCity`, `foundationDate`
- `teamColors`, `userCount`, `national`, `gender`, `class`, `primaryTournament`, `pregameForm`
- Optional: `squad`, `events`

### `recordType: "player"`

- `playerId`, `name`, `slug`, `shortName`, `position`, `positionsDetailed`, `jerseyNumber`, `height`, `preferredFoot`
- `dateOfBirth`, `country`, `countryCode`, `teamId`, `teamName`
- `marketValue`, `marketValueCurrency`, `contractUntil`, `userCount`, `sofascoreId`, `gender`
- Optional: `statistics` (per competition and season)

### `recordType: "standingsRow"`

- `teamId`, `teamName`, `teamSlug`, `teamLogoUrl`, `teamCountryCode`
- `position`, `wins`, `draws`, `losses`, `points`, `gamesPlayed`, `goalsFor`, `goalsAgainst`, `goalDifference`
- `tournamentName`, `tournamentId`, `seasonName`, `seasonId`, `groupName`, `tableType`, `promotion`, `avgRating`

### `recordType: "tournament"`

- `tournamentId`, `name`, `slug`, `sport`, `category`, `country`, `countryCode`, `userCount`
- `seasons`, `standings` (when requested), `currentRound` (when standings are requested)

### `recordType: "searchResult"`

- `type`, `id`, `name`, `slug`, `sport`, `sportSlug`, `country`, `countryCode`
- `score`, `teamColors`, `imageUrl`, `teamId`, `playerId`, `tournamentId`, `eventId`
- Event fields when the result is a match; position, shirt number and market value when it is a player.

### `recordType: "rankingRow"`

- `rankingTypeId`, `rankingTypeName`, `position`, `previousPosition`, `bestPosition`
- `id`, `name`, `teamId`, `teamName`, `teamSlug`, `country`, `countryCode`
- `points`, `previousPoints`, `year`, `tournamentsPlayed`, `playingTeams`, `totalTeams`, `updatedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `live`, `scheduled`, `team`, `player`, `tournament`, `standings`, `event`, `rankings` |
| `searchQueries` | array | – | Names to resolve (mode=search) |
| `searchType` | string | `all` | `all`, `team`, `player`, `tournament`, `event` |
| `sports` | array | `["football"]` | Sports for live/scheduled modes |
| `date` | string | – | Start date for fixtures, `YYYY-MM-DD`; defaults to today UTC (mode=scheduled) |
| `daysAhead` | integer | `0` | Extra days after `date` (0–31) |
| `teamIds` | array | – | Team IDs or URLs (mode=team) |
| `playerIds` | array | – | Player IDs or URLs (mode=player) |
| `tournamentIds` | array | – | Tournament IDs or URLs (modes tournament, standings) |
| `eventIds` | array | – | Match IDs or URLs (mode=event) |
| `seasonId` | integer | – | Optional season for standings/tournament |
| `tableType` | string | `total` | `total`, `home`, `away` |
| `rankingType` | string | `fifa` | FIFA, UEFA, ATP, WTA, UTR, FIP, rugby, UFC or FNC table. SofaScore publishes at most 500 rows per table. |
| `includeStatistics` | boolean | `true` | Match statistics |
| `includeLineups` | boolean | `true` | Lineups and player ratings |
| `includeIncidents` | boolean | `true` | Goals, cards and substitutions |
| `includeOdds` | boolean | `false` | Pre-match odds markets |
| `includeVotes` | boolean | `false` | Crowd prediction polls |
| `includeH2h` | boolean | `false` | Head-to-head tallies |
| `includeAdvanced` | boolean | `false` | Shotmap, momentum, positions, best players, win probability, managers, streaks, pre-game form |
| `includeComments` | boolean | `false` | Text commentary (large payload) |
| `includeSquad` | boolean | `false` | Full squad on team records |
| `includePlayerStats` | boolean | `false` | Per-competition season stats on player records |
| `includeStandings` | boolean | `false` | League table on tournament records |
| `includeEvents` | boolean | `false` | Recent/upcoming events on team records |
| `maxItems` | integer | `50` | Record cap; `0` means no user limit (hard cap 10,000) |

### Example: live football and basketball

```json
{
  "mode": "live",
  "sports": ["football", "basketball"],
  "maxItems": 30
}
```

### Example: a day of fixtures

```json
{
  "mode": "scheduled",
  "sports": ["football", "tennis"],
  "date": "2026-10-28",
  "daysAhead": 1,
  "maxItems": 100
}
```

### Example: a match with all detail

```json
{
  "mode": "event",
  "eventIds": ["13981670"],
  "includeStatistics": true,
  "includeLineups": true,
  "includeIncidents": true,
  "includeOdds": true,
  "includeH2h": true,
  "includeAdvanced": true
}
```

### Example: Premier League table

```json
{
  "mode": "standings",
  "tournamentIds": ["17"],
  "seasonId": 76986,
  "tableType": "total",
  "maxItems": 30
}
```

### Example: FIFA rankings

```json
{
  "mode": "rankings",
  "rankingType": "fifa",
  "maxItems": 200
}
```

## Use cases

- **Live scoreboards** — poll live matches and push score/status updates to a dashboard or alerting pipeline.
- **Fixture calendars** — build daily or multi-day schedules across several sports.
- **Betting research** — combine odds, votes, head-to-head and match statistics in one row per game.
- **Team and player scouting** — pull squads, market values, contracts and season statistics.
- **League monitoring** — track overall, home and away tables over time.
- **Ranking trackers** — follow FIFA, UEFA, ATP, WTA, UTR, FIP, rugby and UFC standings.

## FAQ

### Do I need a SofaScore account, API key or proxy?

No. The actor handles access automatically and needs no login, API key or proxy setup.

### Which sports are supported?

Football, basketball, tennis, American football, Australian rules, ice hockey, handball, rugby, futsal, minifootball, floorball, volleyball, badminton, table tennis, padel, snooker, baseball, cricket, esports, darts, MMA, cycling, motorsport, water polo and bandy. Live coverage varies by time of day.

### Why does `live` return no rows?

There may simply be no match in progress for the selected sports. Use `scheduled` with today's date or `search` to confirm a team exists.

### How do I find a team, player or tournament ID?

Run `mode=search` with the name and use the `id` from the result, or paste the SofaScore page URL directly into `teamIds`, `playerIds`, `tournamentIds` or `eventIds`.

### Why do ranking tables stop at 500 rows?

SofaScore publishes at most 500 rows per ranking table, so `maxItems` above 500 cannot add more. A row's `position` is its place in that table.

### Why are some fields missing?

SofaScore does not publish every field for every match, competition or sport. Missing fields are omitted; they are not replaced with nulls or placeholders.

### Are scores live?

`live` returns the current score and status at the moment the actor runs. It polls SofaScore; it is not a push/streaming feed.

### Can I get historical fixtures?

Yes. Use `scheduled` with a past `date` and `daysAhead: 0`.

### Why do season IDs change?

SofaScore creates a new season ID each year. The actor resolves the latest season automatically for standings and tournaments; pass `seasonId` only when you need a specific historical season.

### Is this affiliated with SofaScore?

No. This is an independent third-party actor that reads publicly available SofaScore data. It is not affiliated with, endorsed by or sponsored by SofaScore.
