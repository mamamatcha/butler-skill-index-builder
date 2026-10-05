# butler-skill-index-builder

The `index-builder` skill for [Butler](https://github.com/Virtual-Protocol/butler-skills).
This README is for the humans maintaining the repo; it is never published to a butler.
Only `SKILL.md` and `references/**/*.md` reach one.

## What it does

Teaches the butler to build an owner a token index: settle the basket, the weights,
the money and the drift band, resolve every ticker to an exact contract, then file one
duty from the `index-rebalance` template. The skill never trades on its own — all of
the money moves inside the duty.

It pairs with
[butler-skill-index-rebalance](https://github.com/Virtual-Protocol/butler-skill-index-rebalance),
which must be listed in the registry's `templates.json` before this skill is listed —
without it, step 4 has nothing to file.

## Layout

```
SKILL.md                the playbook (published)
references/presets.md   starting baskets by theme (published)
README.md               this file (never published)
CHANGELOG.md            one entry per version (never published)
```

## Validating a change

```sh
curl -sSLO https://virtual-protocol.github.io/butler-skills/tools/validate.py
python3 validate.py --standalone .
```

CI runs the same check on every push.

## Releasing

Bump `version` in `SKILL.md`'s frontmatter, add the matching `CHANGELOG.md` entry, and
merge to `main`. A published `name@version` is immutable — the hub's build refuses a
version it already serves with different bytes. No registry PR is needed for a new
version; listing the skill in the first place is maintainer-only.
