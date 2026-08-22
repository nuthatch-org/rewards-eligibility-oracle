# rewards-eligibility-oracle

A [nuthatch](https://github.com/nightswatchhq/nuthatch) nest: **GIP-0088's Rewards Eligibility Oracle on Arbitrum One**.

Authorised oracles mark indexers as eligible to receive indexing rewards. This is the whole record of that, from the contract's first block.

One binary, one config file, no graph-node, no gateway, no query fees.

## What it indexes

**Chain:** `arbitrum-one`. **1 contract**, **9 tables**.

| alias | address |
|---|---|
| `reo` | `0x02753bae61c08abd4351bce7f48524935c2cc78e` |

Instance **A** of an A/B pair, from `packages/issuance/addresses.json` in `graphprotocol/contracts`, chain 42161. B is deployed at `0xeebc4919a239c1315a7e0652e692812719bad591` with code and **zero** logs, which is what an A/B pair looks like when only A is running.

## Verified

Indexed from deployment, blocks **486,290,542 to 497,285,856**, and sealed **1,093 events**:

| table | rows |
|---|---:|
| `reo__indexer_eligibility_renewed` | 1,022 |
| `reo__indexer_tracking_updated` | 50 |
| `reo__indexer_eligibility_data` | 21 |
| `reo__eligibility_validation_updated` | **0** |

## Deployed and emitting, but not yet activated

That last row is the point. `EligibilityValidationUpdated` fires when eligibility validation is switched on, and **it has never fired**. Corroborated two other ways: `RewardsManager.rewardsEligibilityOracle()` reverts, so the oracle is not wired into the rewards flow, and the contract's own source initialises `eligibilityValidationEnabled = false`, *"to be enabled later when the oracle is ready"*.

So this nest indexes a live, real record of a mechanism that is **not yet enforcing**. The oracle has been renewing indexer eligibility since block 492,356,872 regardless. Index it now and you hold the history from the day it starts to count, rather than starting a backfill on the day somebody notices it matters.

## The proxy will lie to you

Both Sourcify and Blockscout answer for `0x02753bae…` with the **proxy's** ABI, which has no events worth indexing. Resolve that address directly and you get a nest that decodes nothing and reports a healthy run.

The ABI here is vendored from the implementation, `0x66cebbec74d413ce45b85156dcb3d7a24724b95b`, read off the EIP-1967 slot.

## What this is not

It is not the whole of the `qos-reo` catalogue entry. That asks for two things, and this is one of them. The other is **gateway quality-of-service telemetry**, which is published off-chain by the gateway rather than by a contract, so no amount of indexing reaches it.

## Run it

```sh
nuthatch init --from https://github.com/nightswatchhq/rewards-eligibility-oracle
cd rewards-eligibility-oracle
nuthatch dev --dir . --backfill 11000000 --seal-direct --window 20000
nuthatch sql --dir . "SELECT count(*) FROM \"reo__indexer_eligibility_renewed\""
```

The endpoint in `nuthatch.toml` is keyless and public, so this file is publishable: a `nuthatch.toml` is pinned into the nest's content address and must never carry a credential. It is enough to follow the tip. A **backfill** wants archive depth it may not have: pass your own with `--rpc`, and check it first with `nuthatch doctor --rpc <url>`.

## Tables

```
reo__indexer_eligibility_renewed
reo__indexer_eligibility_data
reo__indexer_tracking_updated
reo__eligibility_validation_updated
reo__eligibility_period_updated
reo__indexer_retention_period_set
reo__oracle_update_timeout_updated
reo__paused
reo__unpaused
```
