# ESPN Scores, Teams, Players, News & Odds

Scrape **ESPN** for live and historical scores, full game boxscores, league tables, team directories, rosters, player stats, schedules, college and individual-sport rankings, news, and the primary sportsbook line. Ten modes cover 36 leagues across football, basketball, baseball, hockey, soccer, tennis, golf, MMA and Formula 1 — all from public JSON endpoints with **no key, login, cookie or proxy setup**.

Every row carries a `recordType`, so mixed datasets stay easy to filter: `game`, `event`, `gameDetail`, `standing`, `team`, `teamDetail`, `player`, `article`, `scheduleGame`, `ranking`, `oddsLine` and `notice`.

## What this actor does

- **Ten modes:** `scoreboard`, `gameDetail`, `standings`, `teams`, `teamDetail`, `news`, `players`, `schedule`, `rankings`, `odds`
- **36 leagues:** US major leagues, college sports, 19 soccer competitions, ATP/WTA, PGA/LPGA, UFC/PFL and F1
- **Dates and ranges:** today, tomorrow, yesterday, last/next seven days, or a custom range of up to 366 days, fetched one date per request
- **Multi-league runs:** combine leagues in one scoreboard, odds, standings or news run
- **Deep game detail:** team stats, optional player boxscore, leaders, officials, venue, injuries, series, scoring plays, win probability and news
- **Player coverage:** team rosters (flat and grouped), athlete search, profiles, latest-season stats, game logs and rankings
- **Primary odds:** provider, spread, total and moneyline from the first line ESPN publishes
- **Clean output:** empty fields are omitted at every depth; no nulls or placeholder values

## Supported leagues

| Family | League slugs |
|---|---|
| Football | `nfl`, `college-football`, `ufl` |
| Basketball | `nba`, `wnba`, `mens-college-basketball`, `womens-college-basketball` |
| Baseball | `mlb`, `college-baseball` |
| Hockey | `nhl` |
| Soccer | `usa.1`, `eng.1`, `esp.1`, `ger.1`, `ita.1`, `fra.1`, `ned.1`, `por.1`, `mex.1`, `bra.1`, `arg.1`, `uefa.champions`, `uefa.europa`, `uefa.europa.conf`, `eng.fa`, `eng.league_cup`, `fifa.world`, `fifa.cwc`, `concacaf.champions` |
| Tennis | `atp`, `wta` |
| Golf | `pga`, `lpga` |
| MMA | `ufc`, `pfl` |
| Racing | `f1` |

Mode coverage follows the data ESPN publishes: team leagues support every team mode; F1 supports scoreboard, teams, team detail and driver/constructor standings; tennis, golf and MMA return event records with matches, leaderboards or fight cards instead of home/away fields.

## Output

### Scoreboard and odds

Team games include `eventId`, `name`, `shortName`, `url`, `date`, `dateLocal`, `seasonYear`, `seasonType`, `week`, `status`, `statusState`, `statusCompleted`, `venue`, `venueCity`, `venueState`, `venueCountry`, `venueIndoor`, `broadcast`, `broadcasts`, `attendance`, `headlines`, `notes`, `homeTeam`, `homeTeamId`, `homeAbbr`, `homeLogo`, `homeScore`, `homeWinner`, `homeRecord`, `homeForm`, `awayTeam`, `awayTeamId`, `awayAbbr`, `awayLogo`, `awayScore`, `awayWinner`, `awayRecord`, `awayForm`, `competitors[]` and `odds`.

`odds` contains `provider`, `details`, `spread`, `overUnder`, `homeMoneyLine`, `awayMoneyLine`, `homeSpreadOdds`, `awaySpreadOdds`, `overOdds`, `underOdds`, `favoriteTeamId` and `underdogTeamId` when ESPN publishes them. The `odds` mode returns the same line as its own `oddsLine` row.

Tennis rows add `groupings[]` and a flat `matches[]` list with `matchId`, `round`, `court`, `notes`, `status` and `competitors[]`. Golf rows add a ranked `competitors[]` leaderboard with `score` and round-by-round `rounds[]`, plus `leader`, `leaderScore` and `competitorCount`. UFC/PFL rows add `fights[]` with weight class, winner and fighter records. F1 rows add `raceName`, `raceDate`, `circuitName` and a `drivers[]` classification.

### Game detail

`gameDetail` includes the scoreboard identity fields plus `statusState`, `attendance`, `officials[]`, `teamStats[]`, `leaders[]`, `odds`, `winProbability[]`, `predictor`, `recentForm[]`, `scoringPlays[]`, `injuries[]`, `series`, `news[]`, `videos[]`, `recap` and `links`. `includePlayerStats` adds `playerStats[]` with per-athlete stat lines; `includePlayByPlay` adds the complete `plays[]` list.

### Standings

Standing rows keep `team`, `teamId`, `abbreviation`, `conference`, `wins`, `losses`, `winPct`, `rank`, `division`, `gamesBehind` and `pointDifferential`, and add `groupName`, `groupType`, `seasonYear`, `overall`, `gamesPlayed`, `ties`, `points`, `pointsFor`, `pointsAgainst`, `streak`, `homeRecord`, `awayRecord`, `divisionRecord`, `conferenceRecord`, `playoffSeed`, `rankChange`, `logoUrl`, `url` and a complete `stats` map.

F1 driver rows use `athleteId`, `athleteName` and `championshipPoints`; constructor rows use `team`, `teamId` and `championshipPoints`.

### Teams

Team rows include `teamId`, `uid`, `name`, `shortDisplayName`, `abbreviation`, `location`, `color`, `alternateColor`, `slug`, `isActive`, `isAllStar`, `logoUrl`, `darkLogoUrl`, `url`, `rosterUrl`, `competitors`-style `links`, and — when details are fetched — `recordSummary`, `overallRecord`, `homeRecord`, `awayRecord`, `divisionRecord`, `conferenceRecord`, `wins`, `losses`, `winPct`, `gamesPlayed`, `pointsFor`, `pointsAgainst`, `standingSummary`, `conference`, `division`, `venueName`, `venueCity`, `venueState`, `venueIndoor`, `venueLocation`, `nextEvent` and `recordItems[]`.

### Players

Player rows include `athleteId`, `playerId`, `name`, `firstName`, `lastName`, `shortName`, `jersey`, `position`, `positionAbbr`, `age`, `dateOfBirth`, `height`, `displayHeight`, `weight`, `displayWeight`, `birthPlace`, `college`, `collegeId`, `headshot`, `status`, `experienceYears`, `displayExperience`, `debutYear`, `draft`, `teamId`, `teamName`, `teamAbbr`, `salary`, `contractYearsRemaining`, `contractSeason`, `statsSummary` and `url`. Search rows also carry `source: "search"` and `description`; ESPN search is global, so each result keeps the athlete's own `sport` and `league` even when you search from a different league. `stats[]` and `gameLog[]` are attached when the matching toggles are enabled; season stats are published for NBA, NFL, MLB and NHL, while game logs also work for soccer.

### News

Articles include `articleId`, `headline`, `description`, `publishDate`, `lastModified`, `url`, `imageUrl`, `imageCaption`, `imageCredit`, `author`, `category`, `type`, `section`, `source`, `teams[]`, `athletes[]`, `leagues[]`, `gameId`, `videoUrl`, `premium` and `links`. Set `newsScope: "global"` to page the wider ESPN feed; the actor filters those pages down to the selected league.

### Schedule

`scheduleGame` rows combine scoreboard-style fields with `teamId`, `team`, `teamAbbr`, `teamScore`, `teamRecord`, `homeAway`, `opponent`, `opponentId`, `opponentAbbr`, `opponentLogo`, `opponentScore`, `opponentRecord`, `forTeamResult` (`W`, `L` or `D`), `seasonType`, `seasonYear` and `ticketsAvailable`.

### Rankings

Ranking rows include `rankingId`, `rankingName`, `rankingShortName`, `rankingSourceType`, `rank`, `previousRank`, `trend`, `points`, `firstPlaceVotes`, `recordSummary`, `defenses`, `athleteId`, `athleteName`, `team`, `teamId`, `abbreviation`, `logoUrl`, `url`, `date`, `lastUpdated`, `occurrence` and `seasonYear`.

### Notices

When a request is valid but ESPN has nothing to return, or a mode does not apply to a league, the actor pushes a `notice` row with `code`, `message`, `mode`, `league`, `sport` and `hint`. The run still succeeds.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `news` | One of the ten modes |
| `league` | select | `nba` | League slug; the sport is derived automatically |
| `leagues` | array | – | Extra leagues for multi-league runs |
| `datePreset` | select | `today` | `today`, `tomorrow`, `yesterday`, `last7Days`, `next7Days`, `custom` |
| `startDate` | string | – | First date for `custom`, as `YYYYMMDD` or `YYYY-MM-DD` |
| `endDate` | string | – | Last date for `custom`, inclusive; maximum 366 days |
| `season` | integer | – | Season year for standings, schedule, players and rankings; blank uses the current season |
| `seasonType` | select | – | `preseason`, `regular`, `postseason` (schedule; ignored for soccer) |
| `teamIds` | array | – | Team IDs, abbreviations or `league/id` references |
| `eventIds` | array | – | Event IDs or `league/eventId` references |
| `athleteQueries` | array | – | Player names to search |
| `athleteIds` | array | – | ESPN athlete IDs for direct profiles |
| `rankingType` | select | `all` | `all`, `ap`, `coaches`, `cfp`, `pound-for-pound` |
| `statusFilter` | select | `all` | `all`, `pre`, `in`, `post` |
| `liveOnly` | boolean | `false` | Keep only in-progress games |
| `newsScope` | select | `league` | `league` or the paginated `global` feed |
| `includeTeamDetails` | boolean | `true` | Fetch team record, venue and next event |
| `includeGameDetails` | boolean | `false` | Attach full summaries to scoreboard games |
| `includePlayerStats` | boolean | `false` | Add player stats / player boxscore |
| `includeGameLog` | boolean | `false` | Add a player's current-season game log |
| `includePlayByPlay` | boolean | `false` | Add every play to `gameDetail` |
| `maxItems` | integer | `50` | Hard cap (1–5000); notices are metadata rows |

### Example: scoreboard for two leagues

```json
{
  "mode": "scoreboard",
  "leagues": ["nba", "nfl"],
  "datePreset": "yesterday",
  "statusFilter": "post",
  "maxItems": 40
}
```

### Example: full NBA game detail

```json
{
  "mode": "gameDetail",
  "league": "nba",
  "eventIds": ["401909904"],
  "includePlayerStats": true,
  "includePlayByPlay": false,
  "maxItems": 1
}
```

### Example: Premier League schedule and standings

```json
{
  "mode": "schedule",
  "league": "eng.1",
  "teamIds": ["eng.1/364"],
  "seasonType": "regular",
  "maxItems": 38
}
```

```json
{
  "mode": "standings",
  "league": "eng.1",
  "maxItems": 20
}
```

### Example: player search with stats

```json
{
  "mode": "players",
  "league": "nba",
  "athleteQueries": ["LeBron James", "Jannik Sinner"],
  "includePlayerStats": true,
  "maxItems": 10
}
```

### Example: F1 race and driver standings

```json
{
  "mode": "scoreboard",
  "league": "f1",
  "datePreset": "yesterday",
  "maxItems": 5
}
```

```json
{
  "mode": "standings",
  "league": "f1",
  "maxItems": 30
}
```

## Use cases

- **Fantasy and betting dashboards** — pull slates, primary lines and player boxscores into one normalized feed
- **Media and newsroom bots** — monitor league headlines or the global ESPN feed and filter to the teams you cover
- **Team and league trackers** — follow schedules, results, standings and roster changes across multiple leagues
- **Sports research** — build historical datasets from date ranges and full game summaries
- **Player scouting** — search athletes, compare latest-season stats and inspect game logs
- **Broadcast graphics** — feed scoreboards, rankings and leaderboards into live displays

## FAQ

**Do I need an API key, account or proxy?**
No. The actor reads ESPN's public JSON feeds anonymously. There is no credential or proxy input.

**Why did I get a notice row instead of games?**
A notice explains the empty result: no games on that date, no odds published yet, an off-season schedule, or a mode ESPN does not support for that league. The notice `code` and `hint` tell you what to change.

**How many news articles can I get per league?**
ESPN's league news endpoint returns at most 50 articles. Use `newsScope: "global"` to page the wider feed; the actor filters those pages client-side to the selected league.

**Why are some odds fields missing?**
ESPN publishes a primary line for many upcoming games but not every game, and moneyline/spread components vary by sport and provider. Empty odds fields are omitted rather than guessed. The actor ships the first published line only; it does not promise open-line history or every sportsbook.

**Can I scrape tennis, golf, MMA or F1 game summaries?**
Their scoreboards are fully supported with matches, leaderboards, fight cards and driver classifications. ESPN does not expose the same team-style summary for these sports, so `gameDetail` returns a notice for them.

**Why is a team venue missing?**
ESPN only includes venue details for some teams (for example, many NBA/NFL clubs). Soccer and F1 team endpoints commonly omit a home venue. The field is omitted when the source has no value.

**Can I fetch historical seasons and dates?**
Yes. Scoreboard and odds accept a custom date range of up to 366 days. Standings, schedules, players and rankings accept a `season` year. Coverage depends on what ESPN keeps for each league.

**Does the actor return duplicate rows?**
No. Rows are de-duplicated by their natural key (league + event, article, team, athlete or ranking), even when several leagues or player sources overlap.

**Is this affiliated with ESPN?**
No. This is an independent actor that reads publicly accessible ESPN data and is not endorsed by or affiliated with ESPN, Inc.
