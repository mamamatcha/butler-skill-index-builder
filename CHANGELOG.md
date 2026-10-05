# Changelog

## 1.0.0

- First release. Settles a basket of 2-15 tokens and their weights with the owner,
  resolves every ticker to an exact contract, checks liquidity, and files one
  `index-rebalance` duty that holds the basket at those weights.
- Covers checking on an index, changing one, and stopping it.
