---
description: Use cheatcodes to manipulate blockchain state, test reverts, and verify events in your smart contract tests.
---

## Cheatcodes

Most of the time, simply testing your smart contracts outputs isn't enough. To manipulate the state of the blockchain, as well as test for specific reverts and events, Foundry is shipped with a set of cheatcodes.

Cheatcodes allow you to change the block number, your identity, and more. They are invoked by calling specific functions on a specially designated address: `0x7109709ECfa91a80626fF3989D68f67F5b1DD12D`.

You can access cheatcodes easily via the `vm` instance available in Forge Standard Library's `Test` contract. Forge Standard Library is explained in greater detail in the following [section](/forge/tests/forge-std).

Let's write a test for a smart contract that is only callable by its owner.

```solidity
// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:prelude]

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:contract]

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:contract_prelude]

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:simple_test]
}
```

If we run `forge test` now, we will see that the test passes, since `OwnerUpOnlyTest` is the owner of `OwnerUpOnly`.

```sh
forge test
// [!include ~/snippets/output/cheatcodes/forge-test-simple:output}]
```

Let's make sure that someone who is definitely not the owner can't increment the count, by using the `expectRevert` cheatcode:

```solidity
// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:contract_prelude]

    // ...

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:test_expectrevert]
}
```

If we run `forge test` one last time, we see that the test still passes, but this time we are sure that it will always fail if we revert for any other reason.

```bash
forge test
// [!include ~/snippets/output/cheatcodes/forge-test-cheatcodes-expectrevert:output]
```

Another cheatcode that is perhaps not so intuitive is the `expectEmit` function. Before looking at `expectEmit`, we need to understand what an event is.

Events are inheritable members of contracts. When you emit an event, the arguments are stored on the blockchain. The `indexed` attribute can be added to a maximum of three parameters of an event to form a data structure known as a "topic." Topics allow users to search for events on the blockchain.

```solidity
// [!include ~/snippets/projects/cheatcodes/test/EmitContract.t.sol:all]
```

When we call `vm.expectEmit(true, true, false, true);`, we want to check the 1st and 2nd `indexed` topic for the next event.

The expected `Transfer` event in `test_ExpectEmit()` means we are expecting that `from` is `address(this)`, and `to` is `address(1337)`. This is compared against the event emitted from `emitter.t()`.

In other words, we are checking that the first topic from `emitter.t()` is equal to `address(this)`. The 3rd argument in `expectEmit` is set to `false` because there is no need to check the third topic in the `Transfer` event, since there are only two. It does not matter even if we set to `true`.

The 4th argument in `expectEmit` is set to `true`, which means that we want to check "non-indexed topics", also known as data.

For example, we want the data from the expected event in `test_ExpectEmit` - which is `amount` - to equal to the data in the actual emitted event. In other words, we are asserting that `amount` emitted by `emitter.t()` is equal to `1337`. If the fourth argument in `expectEmit` was set to `false`, we would not check `amount`.

In other words, `test_ExpectEmit_DoNotCheckData` is a valid test case, even though the amounts differ, since we do not check the data.

<br></br>

:::info
See the [Cheatcodes Reference](/reference/cheatcodes/overview) for a complete overview of all the available cheatcodes.
:::
