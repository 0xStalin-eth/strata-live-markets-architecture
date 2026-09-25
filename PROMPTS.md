# Regenerating these diagrams

Everything in this repo was produced by Claude Code reading the Strata protocol repository.
This file contains the prompts and the setup needed to reproduce it from a fresh session.

Use it to **refresh the diagrams after the protocol changes**, or to re-derive them
independently as a check on the originals.

---

## 1. Prerequisites

**Clone the protocol repository and check out the exact commit.** The diagrams describe a
specific point in history; running against a different commit gives different — not wrong —
answers.

```bash
git clone https://github.com/Strata-Markets/contracts.git strata-contracts
cd strata-contracts
git checkout 5f77ad4d587450c59dde0afb798a107e24004424
```

| | |
|---|---|
| Commit | `5f77ad4d587450c59dde0afb798a107e24004424` |
| Short | `5f77ad4` — *"add (RiskPremium) use external contract for the calculation (#50)"* |
| Date | 2026-08-29 |
| Branch | `tranches` (the repo's default branch) |
| Mirror | `Strata-Money/contracts-tranches` — public, but lags behind |

**Start Claude Code from the repository root** — the directory containing `contracts/`,
`deployments/` and `src/`. Every path in the prompts is relative to it.

**No build required.** The analysis reads committed JSON and Solidity source only. You do not
need `npm install`, `forge build`, artifacts or an RPC endpoint. `jq` and `python3` are useful
but Claude will manage without them.

---

## 2. Session setup

| Setting | Value | Why |
|---|---|---|
| Model | **Claude Opus 5** | The work is cross-referencing ~100 contracts against deployment records and reconciling contradictions. Sonnet produces plausible diagrams that miss the cooldown gaps. |
| Effort | **xhigh** | Set with `/config`. The verification steps are where the value is, and they reward deeper reasoning. |
| Skills | `artifact-design`, `artifact-diagramming` | Prompt 4 asks for these explicitly. They govern palette, typography and SVG mechanics. |

Run the prompts **in order, in one session**. Prompts 1–3 build the factual ground that
prompt 4 draws from; skipping them yields confident, wrong diagrams.

---

## 3. The prompts

Copy each block verbatim. Wait for each to finish before sending the next.

### Prompt 1 — Establish ground truth

```
This repo is the Strata tranches protocol. Read deployments/*/deployments-eth.json and
src/platforms/Tranches.ts (plus src/platforms/strats/*.ts) and build me a per-market
inventory of everything deployed to Ethereum mainnet.

For each market I want: base asset, StrataCDO address, which accounting contract
(Accounting / DiscreteAccounting / DYSAccounting), the strategy contract, the APR pair
provider, the tranche symbols, and which cooldown contracts exist.

Ignore the .layout.json files. Treat the deployment records as authoritative over the
TypeScript config where they disagree, and tell me if they do.
```

### Prompt 2 — Verify topology (do not skip)

```
For every deployed market, check whether the strategy is single or multi-strategy.
Grep the actual Solidity contract declarations — I want to see whether each reads
"contract X is Strategy" or "is MultiStrategy".

Then tell me where MultiStrategy and IsolatedStrategy are actually used, and whether
any of those markets have a deployment folder.

Cross-check your answer: count StrataCDO vs strategy contracts across every deployment
file on every network. A multi-strategy market would break a 1:1 ratio.
```

### Prompt 3 — Cooldown wiring

```
There are three cooldown contracts: SharesCooldown, ERC20Cooldown, UnstakeCooldown.

For each market work out which are deployed, and — separately — which are actually
wired and functional. Specifically check:
  - which is attached to the CDO vs to the strategy (read the initialize() signatures
    and cdo.setSharesCooldown)
  - whether a CooldownRequestImpl is registered per token (ensureUnstakeImplemenetations)
  - whether the strategy's initialize() even receives the unstake cooldown address

I expect at least one market where a cooldown is deployed but inert. Flag those
explicitly rather than counting them as present.
```

### Prompt 4 — Build the artifact

> This is the prompt that produces the page. It calls the Artifact tool and returns a URL.

```
Build me an HTML artifact: one simple, high-level architecture diagram per deployed
market. Load the artifact-design and artifact-diagramming skills first.

Keep every diagram to the SAME shape so markets are comparable:

  [Tranches group: jrXXX / srXXX] --> [StrataCDO] --> [Strategy] --> [External protocol]
                                          |
                                     [Accounting] <-- [AprPairFeed] <-- [Provider]

Plus a dashed "COOLDOWNS ON WITHDRAWAL" cluster below, with three slots always in the
same position (Shares / ERC20 / Unstake), fed by two arrows: "shares" from the CDO and
"tokens" from the strategy. Where a rail is missing or inert, draw the slot as a dashed
greyed box saying why — the gaps should be visible at a glance.

Rules:
- Hand-authored inline SVG, no libraries. Junior = amber, Senior = teal, consistently.
- Where a strategy reaches its position through two hops (a wrapper vault, or a
  deposit/redemption vault pair), stack two external boxes instead of one.
- Where a market has a deposit-side adapter, show it as a dashed box.
- High-level only: no NAV gating, no per-token exit routing, no reconciliation detail.
- One-line caption per figure stating what's distinctive about that market.
- End with a comparison table: market, protocol wrapped, strategy contract, cooldown
  coverage per rail.
```

### Prompt 5 — The multi-strategy contrast figure

> Republishes the **same** artifact, keeping the URL. Must run in the same session as
> prompt 4 — otherwise see §6.

```
Add a final figure for the multi-strategy market that exists in the deployment scripts
but was never deployed, clearly labelled as not deployed.

Show: tranche pair -> CDO -> IsolatedStrategy -> two sub-strategy legs -> their external
venues, with the Rebalancer between the legs, and the same cooldown cluster.

Verify before drawing:
  - which leg is junior and which is senior (configureStrategies)
  - which cooldowns each leg receives in its initialize()
  - the per-tranche deposit and withdraw ordering (the packed order immutables)
  - whether the parent holds any cooldown references at all

Also add a short panel on what the tranches change in this setup vs a single-strategy
market — deployment differences, ordering constraints enforced on-chain, how a
sub-strategy resolves isJrt, and any per-leg/per-tranche controls.
```

---

## 4. Verification checkpoints

A correct run of prompts 1–3 surfaces these. **If your session reports something different,
resolve the disagreement before drawing** — one of the two analyses is wrong.

- Seven markets deployed to mainnet: `ethena`, `neutrl`, `saturn`, `figure`, `mhyper`,
  `mm1usd`, `nestopal`.
- Accounting split: 3 continuous (`Accounting`), 3 discrete (`DiscreteAccounting`),
  1 DYS (`DYSAccounting`).
- **No deployed market is multi-strategy.** Every live strategy declares `is Strategy`.
  The ratio check gives 8 CDOs to 8 strategy contracts across all networks — the eighth
  pair is the Hoodi testnet instance.
- `IsolatedStrategy`, `MultiStrategy`, `Rebalancer` and `SparkUSDCStrategy` appear in **no**
  deployment file on any network. `spkMhyperIso` is additionally excluded by name in
  `tasks/PlatformFactory.ts`.
- **ethena has no `SharesCooldown`** — not deployed at all, so no CDO→shares arrow.
- **nestopal's `UnstakeCooldown` is inert** — the proxy is deployed, but no
  `CooldownRequestImpl` is registered for nOPAL *and* `NestOpalStrategy.initialize()` never
  receives the address (it takes four arguments, not five).
- `mhyper` and `mm1usd` are two instances of the same `MidasStrategy` contract.
- For the isolated market: junior leg is `SparkUSDCStrategy` (ERC20 cooldown only), senior
  leg is `MidasStrategy` (both token rails). The parent holds **no** cooldown references —
  `MultiStrategy_init` takes only owner, ACM, CDO and the allocation floor.

---

## 5. Matching the original styling

Palette, typography and layout come from the `artifact-design` skill and **will vary between
runs**. The information architecture is what's reproducible; the visual identity is not.

To pin it closer, tell Claude to match what's already here:

```
The eight SVGs in svg/ and the existing index.html are the reference output. Match their
palette, typography and diagram layout so the regenerated page is a drop-in replacement.
```

The originals use:

| Role | Hex |
|---|---|
| Junior | `#A9571A` on `#F6ECE2` |
| Senior | `#0F6959` on `#E5F1EE` |
| Core (CDO, accounting) | `#3A41A0` on `#E9EAF6` |
| External protocol | `#8A3F63` on `#F6E9EF` |
| Cooldowns | `#1F6390` on `#E4EFF6` |
| Ink / muted / rule | `#14171F` / `#667180` / `#CFD5DE` |

Fonts: **Archivo** (headings), **IBM Plex Sans** (body), **IBM Plex Mono** (contract names,
labels, addresses). Market diagrams use `viewBox="0 0 940 356"`; the multi-strategy figure
uses `0 0 940 440`.

---

## 6. Pitfalls

**Only prompt 4 publishes.** Prompts 1–3 print to the terminal and create nothing.

**Prompt 5 needs the same session as prompt 4.** It works by rewriting the same local file and
republishing that path, which preserves the URL. From a fresh session it would create a
*second* artifact instead — prepend `Update the existing artifact at <URL> — don't create a
new one.` if you have to split them.

**Don't feed the answers in.** The prompts deliberately omit market names and the known gaps.
Supplying them produces confident diagrams that were never actually verified, which defeats
the purpose of re-running this.

**Deployment records are not the whole truth.** `FigureStrategy.sol` and `NestOpalStrategy.sol`
first appear in the protocol repo *after* their markets' recorded deployment dates, so those
two were deployed from code not yet pushed. If you need certainty about what is running at a
given address, check the Etherscan-verified source rather than git history.

**Regenerate the SVGs too.** `svg/` is extracted from the inline `<svg class="dia">` blocks in
`index.html`, not referenced by it. If you rebuild the page, re-extract rather than letting the
two drift — see `svg/README.md`.
