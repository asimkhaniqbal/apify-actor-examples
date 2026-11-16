# Pump.fun & Solana New Pool Launch Tracker

Pump.fun tokens start on a bonding curve and only get a real trading pool once they "graduate" to a DEX like Raydium. This actor tracks those graduated pools alongside every other newly created or trending Solana token pool, pulling live price, volume, liquidity, and buy/sell transaction counts so you can spot new launches and momentum shifts as they happen.

## What this actor does

- Lists the newest Solana token pools as they're created, across DEXes such as Raydium and Orca
- Surfaces currently trending tokens ranked by volume/activity
- Looks up a specific token or pool by contract address for full market data
- Searches for a token by name or symbol

## Output per token pool

- `pool_address` — the Solana liquidity pool's contract address
- `token_name`, `token_symbol`, `token_address` — the base token's name, ticker, and contract address
- `quote_token_symbol` — the token it's paired against (usually `SOL`)
- `network` — always `solana`
- `dex_name` — the DEX hosting the pool (e.g. `Raydium AMM`)
- `price_usd` — current token price in USD
- `price_change_5m`, `price_change_1h`, `price_change_24h` — percentage price change over each window
- `volume_usd_24h` — trading volume in USD over the last 24 hours
- `liquidity_usd` — the pool's reserve value in USD
- `fdv_usd` — fully diluted valuation in USD
- `transactions_24h_buys`, `transactions_24h_sells` — buy and sell transaction counts over 24 hours
- `pool_created_at` — when the pool was created
- `image_url` — token logo image, when available
- `url` — link to the pool's market page
- `recordType: "solana_token"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `latestTokens` | `latestTokens` / `trendingTokens` / `newPools` / `searchTokens` / `tokenDetails` |
| `query` | string | – | Token name or symbol to search for (mode=searchTokens) |
| `tokenAddress` | string | – | Token or pool contract address (mode=tokenDetails) |
| `sortBy` | string | `pool_created_at` | Accepted for forward compatibility; currently has no effect on result order (see FAQ) |
| `maxItems` | integer | `50` | Maximum number of token records to return (1–1000) |

### Example: newest Solana launches

```json
{
  "mode": "latestTokens",
  "maxItems": 50
}
```

### Example: trending tokens by activity

```json
{
  "mode": "trendingTokens",
  "maxItems": 30
}
```

### Example: search by symbol

```json
{
  "mode": "searchTokens",
  "query": "RETARD",
  "maxItems": 10
}
```

### Example: full detail for one token address

```json
{
  "mode": "tokenDetails",
  "tokenAddress": "ACuZX4asxyqcRd6BTgGBKXJjViUP3kZQuDUQawBapump"
}
```

## Use cases

- **Meme-coin discovery** — catch freshly launched tokens before they trend
- **Rug-pull and risk screening** — monitor liquidity depth and the buy/sell ratio for suspicious patterns
- **DeFi market research** — track overall Solana DEX launch activity over time
- **Trading signal generation** — surface high-volume new pools early for further analysis
- **Portfolio tracking** — look up price and liquidity for specific tokens you hold

## FAQ

**Does this track pump.fun tokens specifically?**
Yes, but only tokens that have "graduated" — created a real trading pool on a DEX such as Raydium. Tokens still on pump.fun's bonding curve, before that pool exists, aren't tracked, because there's no pool-level market data to report yet.

**Why does the actor describe itself as covering "Solana new pool launches" rather than only pump.fun?**
Every graduated pump.fun token shows up as a regular Solana token pool once it has a DEX pool, alongside pools that never originated on pump.fun at all. The actor tracks that whole pool-level feed rather than a pump.fun-only subset.

**How fresh is the data?**
Pool and token data reflects near-real-time on-chain activity; `mode=latestTokens` or `mode=newPools` surface pools created within the last few hours.

**Can I filter or sort by minimum liquidity or volume?**
Not currently — `sortBy` doesn't change the result order in this version; every mode returns results in that endpoint's own natural order (`newPools`/`latestTokens` by creation time, `trendingTokens` by its own trending ranking, `searchTokens` by relevance). Use `maxItems` to control how many results come back, then sort or filter further downstream.

**What's the difference between `trendingTokens` and `newPools`?**
`trendingTokens` ranks by current trading activity/volume regardless of age; `newPools` returns the most recently created pools regardless of how active they are.

**Why are some fields, like `image_url`, sometimes missing?**
A field is only included when the token's listing actually has that data — not every token has a logo image or a fully diluted valuation figure available.

**Is this affiliated with Pump.fun?**
No. This is an independent, third-party actor; it is not affiliated with or endorsed by Pump.fun or any DEX it tracks.
