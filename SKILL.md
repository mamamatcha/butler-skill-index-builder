---
name: index-builder
description: Build your owner a token index that rebalances itself - pick the basket and weights, buy it, and file the index-rebalance duty that keeps it on target.
version: 1.0.0
metadata: {"butler":{"moneyMoving":true,"keywords":["index","make me an index","basket","auto rebalance","rebalance","portfolio","equal weight","agent index","my index"],"requires":{"bins":["bevo-read","bevo-automation"]}}}
---

## When to use

Your owner wants a basket of tokens held at set weights and kept there for them:
"make me an index of the top AI agent tokens", "put $1k into ETH, SOL and VIRTUAL
equally and keep it balanced", "turn what I hold into an index", "how is my index
doing", "add AIXBT to my index", "stop my index".

The rebalancing itself is the `index-rebalance` duty template. This skill settles the
basket with your owner and files that duty. It never trades on its own.

Not this skill: a one-off basket buy with no rebalancing (that is a plain buy per
token), tokenized stocks or perps in the basket (v1 is spot crypto only), or anything
that times the market.

## Before you start

Settle these with your owner, in this order. `references/presets.md` has starting
baskets for owners who ask for a theme rather than naming tokens.

1. **The tokens**, 2 to 15. Every one must end up as one exact contract on one chain,
   or as ETH, BNB or SOL as a chain's own coin. A ticker alone is never enough.
2. **The weights.** Default is equal weight. The owner can give their own percentages.
   Weights must add up to 100.
3. **The money.** Either a fresh amount to invest (`SEED_USD`), or `0` to adopt what
   they already hold of these tokens as the index.
4. **How tight.** The drift band, default 5 points, and how often to check, default
   daily at 09:00 in their timezone.

Read what they hold before proposing anything:

```sh
bevo-read assets
```

Tell your owner plainly, before filing: **the index is their whole holding of each
token.** If they already hold some VIRTUAL on Base and VIRTUAL is in the index, the
duty will sell some of it or top it up with the rest. If that is not what they want,
leave that token out or hold it on another chain.

## Procedure

1. [ADAPT] Resolve every token to an exact contract. One search per ticker:

   ```sh
   bevo-read token-search <TICKER>
   ```

   Show your owner a card for each ticker listing every match by name, chain and
   address, plus a tile to paste their own address. Ask them to pick even when there
   is only one match, because token-search leaves unverified tokens out. ETH, BNB and SOL
   need only the chain. Never guess a contract.

2. [ADAPT] Check each pick is liquid enough to trade in and out:

   ```sh
   bevo-read token-stats <TICKER>
   ```

   Flag anything with thin liquidity relative to its slice of the index. Every
   rebalance trades it, and a thin pool turns each correction into slippage. Suggest
   dropping it or lowering its weight.

3. [ADAPT] Show the plan and get a clear yes. One message, in this shape:

   ```text
   Index: <name>   <SEED_USD or "your current holdings">
   VIRTUAL (Base)  34%   $340
   AIXBT (Base)    33%   $330
   ETH (Base)      33%   $330
   Rebalances when any weight drifts 5+ pts. Checked daily at 09:00.
   Sells at most $2,000 per rebalance. Trades under $10 are skipped.
   ```

   Repeat the whole-holding warning if any token is already held. Do not file
   anything until the owner says yes to this exact plan.

4. [FIXED] File one duty from the `index-rebalance` template with the duty tool.
   Give it the owner's name for the index as its title, these settings, and a
   `timer` trigger:

   - `LEGS`: one `{SYMBOL, ADDRESS, CHAIN_ID, WEIGHT}` per token, as confirmed in
     step 3. `ADDRESS` is left out only for ETH, BNB or SOL as a chain's own coin.
   - `SEED_USD`, `BAND_PCT`, `MIN_TRADE_USD`, `MAX_REBALANCE_USD` as agreed.
   - Trigger: `{"kind": "timer", "dailyAt": "09:00", "timezone": "<owner's zone>"}`,
     or an hourly interval if the owner asked for a tighter index.

   File it once. The duty buys the starting basket on its first fire.

5. [FIXED] Confirm it is listed:

   ```sh
   bevo-automation list
   ```

### Checking on an index

When your owner asks how their index is doing, read the duty and its recent log,
then their holdings:

```sh
bevo-automation show <INDEX>
bevo-automation logs <INDEX>
bevo-read assets
```

Report each token's current weight against its target, the index's total value, and
the last rebalance from the log. Say whether any trade is waiting on their approval.

### Changing or stopping an index

Adding or dropping a token, or changing weights, the band or the schedule, is an
update to the same duty's settings. Go back through steps 1-3 for anything new, then
update the duty. Never file a second duty for the same basket: two duties would
rebalance against each other.

Stopping is disabling the duty:

```sh
bevo-automation disable <INDEX>
```

Disabling sells nothing. The tokens stay in the owner's wallet. If they want out
entirely, selling each token is a separate request.

## Idempotency and retries

File the duty once per index. If filing errors or you are not sure it went through,
list the duties and look for it. Do not re-run the filing. A second duty on the same
tokens would trade against the first.

The duty keys every trade itself: the starting basket per token, and each rebalance
per scheduled slot and token. A restart never buys or sells twice.

## Failure handling

| Outcome | What to do |
| --- | --- |
| A ticker has no match | Ask the owner for the contract address, or drop the token. |
| Weights do not add to 100 | Show the sum and ask them to fix it. Never rescale silently. |
| Owner has less cash than `SEED_USD` | Say so. Offer a smaller seed, or adopt current holdings. |
| Filing is refused | Report the reason as given and stop. |
| The duty notifies "does nothing until its basket is fixed" | Read the reason it gives, fix that setting with the owner, update the duty. |
| Trades are waiting on approval | Tell the owner. The duty pauses until they approve or decline. |

## Limits

- Spot crypto only in v1: no tokenized stocks, no perps, no cash leg.
- 2 to 15 tokens. Weights are fixed until the owner changes them. Nothing re-picks
  constituents by market cap or momentum automatically.
- The index is the owner's whole holding of each token's exact contract on its chain.
- New money is never added after the seed. Rebalances are funded by their own sells.
- Spending goes through the owner's pocket. Until it is funded, every trade is an
  approval card, and the duty waits on those.
- Not financial advice. Help the owner build the basket they want; do not pick it
  for them as a recommendation.

## Say to the owner

After filing:

"Your <INDEX> index is set up: <N> tokens, rebalanced when anything drifts 5 points,
checked daily at 09:00. The starting basket goes in on the first check. Want me to
send you a note each time it rebalances?"

When checking: lead with total value and the biggest drift, then the per-token table.
