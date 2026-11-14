# UFC Weight-Class Rankings & Fight Card Results

Collect UFC rankings, fighter profiles, event fight cards, and full fight statistics without a login, API key, or proxy. Rankings come from ESPN's public rankings feed; fighter, event, and fight pages come from ufcstats.com, the long-running public record of UFC bouts. Events go back to UFC 2 in 1994.

## What this actor does

- **Five data types:** `rankings`, `fighters`, `fighterDetails`, `events`, `fights`
- **Rankings:** every men's and women's division plus both pound-for-pound lists, with champion and rank-movement markers. With `All Weight Classes`, division lists are emitted first and the pound-for-pound lists last.
- **Fighter directory and profiles:** directory rows carry records and physicals; profiles add date of birth, career striking/grappling rates, and full UFC fight history
- **Events:** completed, upcoming, or both; filter by year and event name; include the full fight card
- **Fight details:** winner, method, round, time, referee, and per-round striking and grappling totals
- **Empty fields are omitted** — a value the source does not publish is left out rather than returned as null

## Output per ranking record (dataType: `rankings`)

- `recordType` — always `fighter`
- `rankingId`, `rankingName` — the ranking list that produced the row
- `rank` — current position
- `isChampion`, `titleDefenses`, `rankTrend` — champion marker, title defences, and movement since the previous list
- `athleteId`, `name`, `nickname`, `weightClass`
- `record`, `wins`, `losses`, `draws`
- `height`, `weight`, `reach`, `stance`, `fightingStyle`, `age`, `dateOfBirth`
- `country`, `headshotUrl`, `espnProfileUrl`, `active`
- `sourceUrl`, `scrapedAt`
- Physical and profile fields appear when `includeFighterBios` is true and ESPN publishes them.

## Output per fighter directory record (dataType: `fighters`)

- `recordType` — always `fighter`
- `athleteId`, `ufcStatsId`, `ufcStatsUrl`
- `firstName`, `lastName`, `name`, `nickname`
- `record`, `wins`, `losses`, `draws`
- `height`, `heightInches`, `weight`, `weightLbs`, `reach`, `reachInches`, `stance`
- `weightClass` — derived from the listed fight weight (for example 155 lb → Lightweight)
- `isChampion` — when the directory marks the fighter as a champion
- `sourceUrl`, `scrapedAt`

## Output per fighter profile (dataType: `fighterDetails`)

- Every directory field above
- `dateOfBirth`, `dateOfBirthIso`, `age`
- `slpm`, `strikingAccuracy`, `sapm`, `strikingDefense` — significant strikes landed/absorbed per minute and accuracy/defence percentages
- `takedownAverage`, `takedownAccuracy`, `takedownDefense`, `submissionAverage`
- `fightCount`, `fightHistory` — array of UFC bouts:
  - `result`, `opponent`, `opponentId`
  - `eventName`, `eventId`, `eventUrl`, `eventDate`, `eventDateDisplay`
  - `method`, `methodCode`, `methodDetail`, `round`, `time`
  - `fightId`, `fightUrl`, `isTitleFight`, `bonuses`
  - `knockdowns`, `significantStrikes`, `takedowns`, `submissionAttempts`

## Output per event (dataType: `events`)

- `recordType` — always `event`
- `eventId`, `ufcStatsEventId`, `ufcStatsUrl`
- `eventName`, `date`, `dateDisplay`, `year`, `location`, `eventStatus`
- `fightCount`, `fights` — array of bouts:
  - `fightId`, `fightUrl`
  - `result`, `winner`
  - `fighter1`, `fighter2`, `fighter1Id`, `fighter2Id`, `fighter1Url`, `fighter2Url`
  - `weightClass`, `isTitleFight`, `bonuses`
  - `method`, `methodCode`, `methodDetail`, `round`, `time`
  - `fighter1Kd`, `fighter1Str`, `fighter1Td`, `fighter1Sub` and the matching `fighter2*` totals
  - With `includeFightStats`: `fighter1Stats`, `fighter2Stats`, `rounds`, `referee`, `scheduledRounds`, `timeFormat`, `fighter1Result`, `fighter2Result`, `loser`, `isDraw`, `isNoContest`
- `sourceUrl`, `scrapedAt`

## Output per fight (dataType: `fights`)

- `recordType` — always `fight`
- `fightId`, `fightUrl`, `eventId`, `eventUrl`, `eventName`
- `isTitleFight`, `weightClass`
- `method`, `methodDetail`, `round`, `time`, `timeFormat`, `scheduledRounds`, `referee`
- `fighter1Name`, `fighter2Name`, `fighter1Nickname`, `fighter2Nickname`
- `fighter1Result`, `fighter2Result`, `winner`, `loser`, `isDraw`, `isNoContest`
- `fighter1Stats`, `fighter2Stats` — totals:
  - `knockdowns`, `sigStrikesLanded`, `sigStrikesAttempted`, `sigStrikeAccuracy`
  - `totalStrikesLanded`, `totalStrikesAttempted`
  - `takedownsLanded`, `takedownsAttempted`, `takedownAccuracy`
  - `submissionAttempts`, `reversals`, `controlTimeSeconds`
  - `headStrikesLanded`, `headStrikesAttempted`, `bodyStrikesLanded`, `bodyStrikesAttempted`, `legStrikesLanded`, `legStrikesAttempted`
  - `distanceStrikesLanded`, `distanceStrikesAttempted`, `clinchStrikesLanded`, `clinchStrikesAttempted`, `groundStrikesLanded`, `groundStrikesAttempted`
- `rounds` — one object per round with the same `fighter1Stats` and `fighter2Stats`
- `sourceUrl`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `dataType` | string | `rankings` | `rankings`, `fighters`, `fighterDetails`, `events`, or `fights` |
| `weightClass` | string | `Lightweight` | Division for rankings, or `All Weight Classes` |
| `searchQuery` | string | – | Fighter name or nickname (dataType=`fighters` or `fighterDetails`) |
| `fighterIds` | array | – | Fighter IDs or fighter-details URLs (dataType=`fighterDetails`) |
| `fighterLetter` | string | `All letters` | Last-name initial for the fighter directory |
| `stance` | string | `Any stance` | Orthodox, Southpaw, Switch, Open Stance, or Sideways |
| `minWins` | integer | – | Minimum recorded wins (dataType=`fighters`) |
| `sortFighters` | string | `name` | `name`, `wins`, `losses`, or `bouts` |
| `eventType` | string | `completed` | `completed`, `upcoming`, or `all` |
| `eventSearch` | string | – | Case-insensitive event-name filter |
| `eventSort` | string | `newest` | `newest` or `oldest` |
| `season` | integer | – | Event calendar year, 1994–2035 |
| `eventIds` | array | – | Specific event IDs or event-details URLs |
| `includeFightCards` | boolean | `true` | Fetch each event page for its fight card |
| `includeFightStats` | boolean | `false` | Add round-by-round stats to every fight in an event |
| `includeFightHistory` | boolean | `true` | Add fight history to fighter profiles |
| `includeFighterBios` | boolean | `true` | Add physical stats to ranking rows |
| `fightIds` | array | – | Fight IDs or fight-details URLs (dataType=`fights`) |
| `maxItems` | integer | `50` | Hard cap on emitted records (1–5000) |

Older saved tasks that send `mode` keep working: `topRanked`, `byEvent`, and `searchFighters` are mapped to the matching data type automatically. A legacy `byEvent` task keeps its current-year default when `season` is omitted.

### Example: current lightweight rankings

```json
{
  "dataType": "rankings",
  "weightClass": "Lightweight",
  "maxItems": 10
}
```

### Example: southpaw fighters with at least 10 wins

```json
{
  "dataType": "fighters",
  "fighterLetter": "m",
  "stance": "Southpaw",
  "minWins": 10,
  "sortFighters": "wins",
  "maxItems": 25
}
```

### Example: two fighter profiles with fight history

```json
{
  "dataType": "fighterDetails",
  "fighterIds": [
    "032cc3922d871c7f",
    "http://ufcstats.com/fighter-details/f4c49976c75c5ab2"
  ],
  "includeFightHistory": true
}
```

### Example: 2024 events with fight cards

```json
{
  "dataType": "events",
  "eventType": "completed",
  "season": 2024,
  "eventSort": "oldest",
  "includeFightCards": true,
  "maxItems": 3
}
```

### Example: full stats for a title fight

```json
{
  "dataType": "fights",
  "fightIds": ["3eea92e7f26fb05b"]
}
```

## Use cases

- **MMA betting and fantasy models** — feed current rankings, recent results, and per-round strike/takedown totals into fight predictors
- **Fight databases** — build a historical UFC archive from UFC 2 onward with one row per event and one row per bout
- **Content and newsletters** — generate fight-card previews, result recaps, and division movement reports
- **Athlete research** — compare career striking and grappling rates across fighters
- **Sports analytics** — study how finish methods, control time, and striking volume change by division and era

## FAQ

**Does this need an API key, login, or proxy?**
No. Rankings are read from ESPN's public feed, and ufcstats.com pages are read directly without an account, cookie input, or proxy.

**Which promotions are covered?**
The fight data covers UFC events, including the early UFC tournaments. Other promotions that appear in a fighter's pre-UFC record are not part of the fight history tables.

**Why is a champion ranked #1 even when the division list shows a lower rank?**
ESPN publishes champion-only groups alongside the division lists. The actor merges those groups; when the same champion also appears lower in the division list, the champion group's rank 1 wins and the row is marked `isChampion`.

**How many fighters appear in each ranking?**
The public ESPN feed usually publishes 9–10 ranked fighters per division, not a fixed top 15. Both the champion and the ranked contenders are included whenever ESPN lists them. With `All Weight Classes`, division lists come first and the two pound-for-pound lists last.

**Can I get events from any year?**
Yes. Set `season` to a year from 1994 to 2035, or leave it blank to scan the full completed or upcoming calendar.

**Does `events` return the whole fight card?**
Yes when `includeFightCards` is true. Each bout includes both fighters, the winner, method, round, time, and the basic knockdown/strike/takedown/submission totals shown on the event page. `eventStatus` is `upcoming` until results appear; an event dated today turns `completed` only after its fights have results.

**How do I get round-by-round stats?**
Set `includeFightStats` to true in `events`, or use `fights` mode with fight IDs. The actor then reads each fight page and returns totals plus one stat block per round.

**Why is `weightClass` sometimes missing on a fight record?**
A fight page only names a division for a standard or title bout. Catch-weight, open-weight, and tournament bouts may not carry a division label, so the field is omitted instead of guessed.

**Why does a fighter in the directory have a different division than expected?**
Directory rows do not publish a division, so `weightClass` is derived from the listed fight weight. Fighters who change divisions or catch weights will show the weight listed on their profile.

**Are posters, photos, or rankings history included?**
No. The actor returns the current rankings, fighter records and stats, event cards, and fight results. It does not return event posters, image galleries, or past ranking snapshots.

**Is this affiliated with the UFC or ESPN?**
No. This is an independent actor that reads publicly available MMA data and is not operated by or affiliated with the UFC or ESPN.
