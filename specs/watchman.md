# Watchman: pre-flight checks for IMD jury mandate actions

Codename *Watchman* (Ezekiel 33). Status: **open**. Last checked: 2026-11-18.

## Pitch
Before an IMD juror attests a treasury action proposed under a Mandate, an open-source tool lints
the action against the Mandate and simulates it on a fork. It prints a deterministic verdict that
jurors can compare with each other. It advises; it never attests, signs or executes.

## What we know (and how)
- Verified: [Identity-md/worker releases](https://github.com/Identity-md/worker/releases) 0.1.32 to
  0.1.37 (Nov 15-17) add the jury: juror keys under a separate `imd-juror` user, jailed tasks,
  mandate-capped gas-station tips.
- Verified: a public report of the first jury mandate action on Sepolia (kind `treasury.rebalance`,
  quorum 2, queued through a Safe Delay module):
  [report](https://github.com/Identity-md/research/blob/main/jobs/29dbb6b4-60d7-4de9-b0a8-9f850a79b9ad/files/artifacts/goal/writeup/report.md).
- **Inferred** (from worker strings and the report, not public docs): a Mandate is scoped to one
  chain, lists allowed recipients, spenders and functions, and lasts at most 90 days. Treat the
  schema as a guess until IMD documents it.

## Milestone 1 (local, anvil fork of Sepolia)
1. Mandate schema v0 (JSON) from the public report; mark every inferred field.
2. Static checks: recipient or spender not allowlisted; function selector not allowlisted;
   unbounded approvals; `delegatecall`; proxy targets (EIP-1967 slots); EIP-7702-delegated EOAs as
   targets; expiry passed; chain id mismatch.
3. Fork simulation (anvil `--fork-url` Sepolia at the proposal block): balance and allowance diff for
   the treasury and every touched address; Safe module path replayed.
4. Verdict: canonical JSON `{action_hash, mandate_hash, block, checks[], diffs[], verdict}`, so two
   jurors running the same version get the same bytes. Optional ed25519 / agb1 signature.
5. Package as a SKILL.md an IMD seat can run.

## Acceptance
- The Sepolia report's action reproduces with a PASS; three mutated fixtures (extra recipient,
  unlimited approve, delegatecall) FAIL with the right check id.
- Deterministic output across two machines.

## Ground rules
Jurors decide; the tool only informs. No mainnet signing. Don't publish anything that looks like a
vulnerability in a live treasury; report those privately to IMD.
