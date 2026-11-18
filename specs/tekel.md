# Tekel: a read-only audit of pump.fun tokenized agents

Codename *Tekel* ("weighed in the balances", Daniel 5:27). Status: **open**. Last checked: 2026-11-18.

## Pitch
For each pump.fun tokenized agent, measure how much of its "revenue" comes from outside customers
versus wallets linked to its creator, and whether the hourly buyback-and-burn matches the
configured percentage. Output a dated, signed, reproducible report. It never trades and never signs
a Solana transaction.

## What we verified (official sources, 2026-11-17/18)
- [pump.fun tokenized-agent disclaimer](https://pump.fun/docs/tokenized-agent-disclaimer): each agent
  has an Agent Deposit Address that **anyone can deposit to** (SOL, USDC, USDT, USD1). A creator-set
  percentage buys back and burns the coin **hourly**, through a Tokenized Agent Authority that Pump
  controls centrally. The rest is creator-claimable.
- Payments SDK: `@pump-fun/agent-payments-sdk` (invoice PDAs over mint, currency, amount, memo,
  start, end). [pump-fun-skills](https://github.com/pump-fun/pump-fun-skills) has a
  `tokenized-agents` skill. [pump-public-docs](https://github.com/pump-fun/pump-public-docs) has the
  program IDLs (`pump`, `pump_amm`, `pump_fees`).
- Because anyone, the creator included, can deposit, deposits alone don't prove customer demand.

## Unverified
- The agent-payments program id and whether its IDL ships in the npm tarball.
- Exact buyback timing rules (which hour window, rounding, minimums).

## Milestone 1 (read-only mainnet RPC, or devnet)
1. Indexer: for a handful of tokenized agents that **opted in** (or a devnet fixture), pull inflows to
   the deposit address and invoice payments, plus buyback and burn txs.
2. Funding lineage: cluster payers by funding source (first funder, shared funders within N hops,
   CEX hot-wallet labels where public). Score `organic_ratio = outside / total`, with a confidence.
3. Buyback check: per hour, `expected = pct * eligible_inflow` vs observed buy + burn; report drift.
4. Report: JSON + Markdown per agent per day, signed (ed25519 / agb1 attestation optional).

## Acceptance
- Re-running on the same slot range gives byte-identical JSON.
- Every number links to the transactions behind it.
- Methodology published; false-positive cases documented.

## Ground rules
Methodology and opt-in agents only in public output. Never accuse: report a ratio and a confidence,
not intent. No sniping, no mempool features, no trading signals.

## Later
D4: prove the score with SP1 and verify on Solana (sp1-solana) or on FLOP (see Proof Court).
