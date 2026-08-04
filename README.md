# Bonded Agent

> An AI Agent's transaction promise should come with collateral.

Bonded Agent is Backchannel's Monad Playground project. It adds an economic commitment layer to AI-assisted swaps: before a user signs, the Agent operator locks collateral for a specific promised outcome. If the completed swap falls below the guaranteed output, the contract pays the shortfall automatically.

## The problem

A simulation tells a user what may happen under a particular chain state. It is not a guarantee: liquidity, prices, and transaction conditions can change before execution. Today, when an Agent's projected result does not materialize, the user usually carries the entire loss.

Bonded Agent turns a narrow, measurable promise into a verifiable onchain commitment.

## Current build

- [Current interface preview](https://bonded-agent-web.vercel.app/)
- The current interface is a demo flow. It honestly labels the market as `MockDex`; it does not yet prove that a live Testnet contract has locked collateral or completed a settlement.
- The P0 target is three verifiable Monad Testnet paths: normal execution, automatic shortfall compensation, and safe failure with refund.

## Demo scope

For the hackathon, we only support one objective scenario:

```text
1 MON -> tUSDC on Monad Testnet
```

The operator publishes a one-time plan containing the user, input amount, expected and guaranteed output, maximum compensation, expiry, target contract, calldata hash, and nonce. The operator locks tUSDC collateral before the user executes.

- If actual output meets the guarantee, collateral is released.
- If actual output is below the guarantee but inside the covered range, the contract pays the exact difference.
- If the result falls outside the covered range or the swap fails, the contract refunds the input and pays the predefined failure compensation.

## Why Monad and Moss

Moss helps an Agent discover an action, prepare an unsigned transaction, simulate the result, and explain it before signing. Bonded Agent asks the next question: who bears the economic cost if that expected result is not delivered?

Monad Testnet is where we deploy and demonstrate real collateral locking, execution, automatic compensation, and event-based verification. Moss integration is a P1 item: the P0 mechanism must work without an LLM or administrator deciding whether a promise was broken.

## What is real vs. simulated

| Component | P0 status |
| --- | --- |
| Interface flow and MockDex quote | Current demo |
| Plan, collateral, execution, settlement, and event evidence | To be deployed on Monad Testnet |
| Normal execution and shortfall compensation cases | To be verified on Monad Testnet |
| Market pricing | Mock DEX with configurable rate, disclosed as simulation |
| Moss workflow | Conditional P1 integration |
| Multi-operator reputation, real DEX, and service fee | Future work |

## Team

| Role | Focus |
| --- | --- |
| Research / Product | Product boundary, promise definitions, testing evidence, pitch |
| Marketing / Operations | User testing, positioning, documentation, demo production |
| Dev 1 | Solidity contracts, tests, Testnet deployment |
| Dev 2 | Agent flow, frontend, wallet and contract integration |
| Design | Promise card, transaction states, user-facing risk explanation |

## Project docs

- [Five-day work plan](docs/five-day-workplan.md)
- [P0 product rules and contract acceptance](docs/product-spec.md)
- [Interface states and copy](docs/interface-state-copy.md)
- [Validation, evidence, and user-test protocol](docs/validation-and-user-test.md)
- [Research / product operating board](docs/researcher-operating-board.md)

## Non-negotiable boundaries

- We do not guarantee profit.
- We do not use an LLM or admin to determine a shortfall.
- We do not present a Mock DEX as a production market.
- We do not claim that the Agent is correct in every possible task.
