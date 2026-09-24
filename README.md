# Strata — Live Markets Architecture

High-level architecture schematics for every Strata CDO market deployed to Ethereum mainnet,
plus the multi-strategy topology that exists in the deployment scripts but was never deployed.

Derived from commit [`5f77ad4`](https://github.com/Strata-Markets/contracts) of the protocol
repository — specifically `deployments/*/deployments-eth.json`, `src/deployments/*.ts`,
`src/platforms/Tranches.ts` and the strategy contracts themselves.

## Viewing it

It is a single self-contained HTML file. No build step, no dependencies, no JavaScript.

**Simplest — open it directly:**

```bash
git clone https://github.com/0xStalin-eth/strata-live-markets-architecture.git
cd strata-live-markets-architecture
open index.html            # macOS
xdg-open index.html        # Linux
start index.html           # Windows
```

**Or serve it locally**, if you prefer a real origin:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

The page renders in your system light/dark theme automatically.

## What's in it

One diagram per market, all drawn to the same shape so they can be compared side by side:

```
[Tranches: jrXXX / srXXX] → [StrataCDO] → [Strategy] → [External protocol]
                                 ↕
                           [Accounting] ← [AprPairFeed] ← [APR provider]

              + COOLDOWNS ON WITHDRAWAL (Shares / ERC20 / Unstake)
```

| Market | Base | Accounting | Strategy | External position |
|---|---|---|---|---|
| ethena | USDe | `Accounting` | `sUSDeStrategy` | sUSDe |
| neutrl | NUSD | `Accounting` | `sNUSDStrategy` | sNUSD |
| saturn | USDat | `Accounting` | `SaturnStrategy` | sUSDat |
| figure | USDC | `DiscreteAccounting` | `FigureStrategy` | wYLDS → PRIME |
| mhyper | USDC | `DiscreteAccounting` | `MidasStrategy` | mHYPER |
| mm1usd | USDC | `DiscreteAccounting` | `MidasStrategy` | mM1-USD |
| nestopal | USDC | `DYSAccounting` | `NestOpalStrategy` | nOPAL |

A final figure covers `spkMhyperIso` — the only market configured as multi-strategy
(`IsolatedStrategy` over a Spark USDC junior leg and a Midas mHYPER senior leg). It has a
complete deployment class and fork tests but appears in no deployment file on any network,
so it is drawn for contrast and labelled as not deployed.

## Notes on accuracy

- Every deployed strategy inherits the single `Strategy` base — none of the seven live
  markets is multi-strategy. Verified from the contract declarations, and cross-checked
  against the deployment records: 8 CDOs to 8 strategy contracts, a strict one-to-one.
- Cooldown coverage is drawn as wired, not merely as deployed. Two markets differ from the
  rest and the diagrams show why rather than hiding it.
- Addresses shown are proxy addresses.

## Licence / status

Architecture documentation compiled from public on-chain deployments and the protocol's own
repository. Not an audit report, and it contains no security findings.

## Diagram sources

The eight diagrams are also available as standalone SVG files under [`svg/`](svg/) — self-contained,
not referenced by the page, provided for editing or embedding in reports. See [`svg/README.md`](svg/README.md).
