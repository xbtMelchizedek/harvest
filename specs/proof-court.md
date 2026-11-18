# Proof Court: agent claims notarized by FLOP's native SP1 verifier

Codename *Urim & Thummim*. Status: **open**. Last checked: 2026-11-18.

## Pitch
An agent proves a claim about itself (a reputation score computed from signed receipts, a job
result, an eval score) with an SP1 zkVM proof, submits it to FLOP's own SP1 verifier pallet, and
gets an on-chain verification event. FLOP becomes a neutral notary for agent claims on the same
chain where agent compute is paid. No bridge and no smart contract.

## What we verified (read-only, 2026-11-18)
- FLOP devnet RPC `https://rpc.devnet.flop.network`: `system_name` = "zkVerify", runtime
  `tzkv-runtime`. The node is built on Horizen Labs' [zkVerify](https://github.com/HorizenLabs/zkVerify).
- Runtime metadata includes `SettlementSp1Pallet` (plus Groth16, RISC Zero, Plonky2, UltraHonk, ...),
  `Aggregate` and `AgentIdentity` (21,336 agents registered on devnet). At the time of the read the
  SP1 pallet had **0 registered vks and 0 tickets**: nobody had used it.
- zkVerify's SP1 pallet ([docs](https://docs.zkverify.io/architecture/verification_pallets/sp1),
  [supported proofs](https://docs.zkverify.io/architecture/supported_proofs)) accepts **SP1 v5.x
  shrink proofs**; vk = `hash_babybear` as 32 LE bytes (H256); public values up to 2048 bytes.
  `submitProof(vkOrHash, proof, pubs, domainId)`. Statement hash = keccak256("sp1"), vk, keccak256(pubs).
- [zkVerify/sp1-verifier](https://github.com/zkVerify/sp1-verifier) pins **SP1 5.0.5** and ships
  `sp1-zkv-sdk::convert_proof_to_zkv` to turn a compressed proof into the pallet format.

## Unverified (find out first)
- Whether FLOP's fork matches upstream zkVerify byte for byte (pallet versions, SP1 version).
- Whether the pallets ship on FLOP testnet (target Nov 24) and whether `submitProof` is open to any
  account or allowlisted. FLOP's public docs don't mention the pallets. Ask Flop (yellowpaper
  "Question" issue) before relying on them.
- Fees and weight for a submission.

## Milestone 1 (local + read-only)
1. Port an SP1 program to **SP1 5.0.5**. Start from
   [sp1-reputation](https://github.com/xbtMelchizedek/sp1-reputation) (`agb-rep`, currently SP1 6.x,
   execute mode only) or a fibonacci hello-world.
2. Produce one compressed proof locally (CPU prover; budget RAM, it is large) and convert it with
   `sp1-zkv-sdk`. Verify it offline with `sp1_zkv_sdk::verify`.
3. Build the `settlementSp1Pallet.submitProof` extrinsic with polkadot.js against devnet metadata.
   Check it only with `payment_queryInfo` and an encode/decode round trip. **Do not sign.**
4. Publish: the proof bytes, vk hash, pubs, and the exact toolchain versions.

## Milestone 2 (devnet, separate devnet key)
Submit on devnet from a throwaway devnet account; record the `ProofVerified` statement hash; index
statement ↔ agent DID ↔ `AgentIdentity`. Post an agb1 receipt that names the statement hash.

## Acceptance
- A reproducible script: same inputs give the same vk hash and pubs.
- A negative test: tampered pubs fail offline verification.
- No mainnet anything; no key material in the repo.

## Risks
SP1 version mismatch (5.x vs 6.x). CPU proving RAM. Pallets may be disabled or allowlisted on
testnet. Proof submissions might be treated as spam: batch and keep it rare.
