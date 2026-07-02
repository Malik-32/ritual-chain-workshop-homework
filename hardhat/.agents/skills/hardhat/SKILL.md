---
name: hardhat
description: Use when working with Hardhat 3 projects — writing or modifying Solidity tests, TypeScript tests, or any code touching hardhat.config.ts, the `hardhat` import, or `network.create()`. Covers test-layer choice, forge-std cheatcodes, the network connection API, `networkHelpers`, and the compile-then-typecheck workflow. For toolbox-specific guidance (clients, contract calls, assertions), also load the matching `hardhat-toolbox-*` skill.
metadata:
  package: "hardhat"
---

# Hardhat 3

This skill covers Hardhat 3 itself. The toolbox layer (clients, contract interaction, ecosystem-specific assertions) lives in companion skills — load whichever matches the project:

- `@nomicfoundation/hardhat-toolbox-viem` → also load the **`hardhat-toolbox-viem`** skill.
- `@nomicfoundation/hardhat-toolbox-mocha-ethers` → also load the **`hardhat-toolbox-mocha-ethers`** skill.

## Test organization

Hardhat 3 supports two distinct test layers:

**Solidity tests** (`.t.sol` files in `contracts/`, or any `.sol` file in `test/`) are the default choice for unit tests on individual contracts. They run directly in the EVM, compile-check against the real ABI, and have access to cheatcodes for EVM state manipulation. Any public function whose name starts with `test` is executed as a test case.

**TypeScript tests** (`.ts` files in `test/`) are the right choice when a test requires off-chain orchestration: multi-contract interactions, fixture reuse across a large suite, assertions about gas or ETH balances from the outside, or integration scenarios driven by external state.

Reach for TypeScript only when Solidity isn't enough. Solidity tests cover all contract logic; TypeScript tests cover the end-to-end flow as a user or script would experience it.

```bash
hardhat test            # run all tests (Solidity + TypeScript)
hardhat test solidity   # Solidity tests only
```

TypeScript tests are run either with `hardhat test nodejs` or with `hardhat test mocha`, depending on the toolbox or plugins being used: the viem-based toolbox uses `nodejs` while the ethers+mocha toolbox uses `mocha`.

Pass `--coverage` to collect Solidity coverage while running tests. It works on the main `hardhat test` task as well as the `solidity` and TypeScript subtasks (`nodejs` / `mocha`), so you can scope coverage to a single test layer when needed.

## Solidity tests

Any contract that contains at least one `test*` function is treated as a test contract. Use `forge-std` (if present in `package.json`) for assertions and cheatcodes:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.28;

import { Test } from "forge-std/Test.sol";
import { Counter } from "./Counter.sol";

contract CounterTest is Test {
  Counter counter;

  function setUp() public {
    counter = new Counter();
  }

  function test_InitialValueIsZero() public view {
    assertEq(counter.x(), 0);
  }
