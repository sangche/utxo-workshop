# UTXO on Substrate

A UTXO chain implementation on Substrate, with two self-guided workshops.Original [UXTO inspiration](https://github.com/0x7CFE/substrate-node-template/tree/utxo) by [Dmitriy Kashitsyn](https://github.com/0x7CFE).

Substrate Version: `2.0.0-alpha.6`. For educational purposes only.

## Table of Contents
- [Installation](#Installation): Setting up Rust & Substrate dependencies

- [UI Demo](#UI-Demo): Demoing this UTXO implementation in a simple UI

- [Beginner Workshop](#Beginner-Workshop): A self guided, 1 hour workshop that familiarizes you with Substrate basics

- [Advanced Workshop](#Advanced-Workshop): A self guided, 2 hour video tutorial, that teaches you how to build this UTXO blockchain from scratch

- [Helpful Resources](#Helpful-Resources): Documentation and references if you get stuck in the workshops


## Installation

### 1. Install or update Rust
```zsh
curl https://sh.rustup.rs -sSf | sh

# On Windows, download and run rustup-init.exe
# from https://rustup.rs instead

rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
rustup update stable
cargo install --git https://github.com/alexcrichton/wasm-gc
```

### 2. Clone this workshop

Clone your copy of the workshop codebase

```zsh
git clone https://github.com/substrate-developer-hub/utxo-workshop.git
```

## UI Demo

In this UI demo, you will interact with the UTXO blockchain via the [Polkadot UI](https://substrate.dev/docs/en/development/front-end/polkadot-js).

The following demo takes you through a scenario where:
- Alice already owns a UTXO of value 1000 upon genesis
- Alice sends Bob a UTXO with value 120, sends herself a UTXO with value 580,
  tipping the remainder to validators

This is multiple TransactionOutputs demo.

1. Compile and build a release in dev mode
```
# Initialize your Wasm Build environment:
./scripts/init.sh

# Build Wasm and native code:
cargo build --release
```

2. Start your node & start producing blocks:
```zsh
./target/release/utxo-workshop --dev

# If you already modified state, run this to purge the chain
./target/release/utxo-workshop purge-chain --dev
```

3. In the console, notice the helper printouts. In particular, notice the default account `Alice` was already has `100 UTXO` upon the genesis block.

4. Open [Polkadot JS](https://polkadot.js.org/apps/#/settings), making sure the client is connected to your local node by going to Settings > General, and selecting `Local Node` in the `remote node` dropdown.

5. **Declare the custom datatypes in PolkadotJS**, since the JS client cannot automatically infer this from the UTXO module. Go to Settings > Developer tab and paste in the following JSON:

```json
{
  "Address": "AccountId",
  "LookupSource": "AccountId",
  "Value": "u128",
  "TransactionInput": {
    "outpoint": "Hash",
    "sigscript": "H512"
  },
  "TransactionOutput": {
    "value": "Value",
    "pubkey": "Hash"
  },
  "Transaction": {
    "inputs": "Vec<TransactionInput>",
    "outputs": "Vec<TransactionOutput>"
  },
  "Difficulty": "U256",
  "DifficultyAndTimestamp": {
    "difficulty": "Difficulty",
    "timestamp": "Moment"
  },
  "Public": "H256",
  "Weight": "u32"
}
```

6. **Confirm that Alice already has 1000 UTXO at genesis**. In `Chain State` > `Storage`, select `utxo`. Input the hash `0xcfc13fe5c575a0cee0ad857a3ea1278aff33e5f67c180359af6bb39bae939bad`. Click the `+` notation to query blockchain state.

    Notice that:
    - This UTXO has a value of `1000`
    - This UTXO belongs to Alice's pubkey. You use the [subkey](https://substrate.dev/docs/en/next/development/tools/subkey#well-known-keys) tool to confirm that the pubkey indeed belongs to Alice
		
```sh
subkey inspect //Alice
```

7. **Spend Alice's UTXO, giving 50 to Bob.** In the `Extrinsics` tab, invoke the `spend` function from the `utxo` pallet, using Alice as the transaction sender. Use the following input parameters(with 2 TransactionOutputs):

    - outpoint: `0xcfc13fe5c575a0cee0ad857a3ea1278aff33e5f67c180359af6bb39bae939bad`
    - sigscript: `0xbc891950e38785baa4954195e750cb6c846d5cbb04bfe8b207931fce6192c76386c1b9fa12bfbcb967029192260194ad20254ed96d00976700c61a7f5c27ae8a`
    - value: `120`
    - pubkey: `0x8eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48`
		- value: `580`
		- pubkey: `0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d`

    Send this as an `unsigned` transaction. With UTXO blockchains, the proof is already in the `sigscript` input.

8. **Verify that your transaction succeeded**. In `Chain State`, look up the newly created 2 UTXO hashes: First one `0xf5171e38f877fae0d04f316586ec2640eac9f1c58ce76e5c91bbc00a6b0d2d0a` to verify that a new UTXO of 120, belonging to Bob;
Second one `0x6b099ef24a3af636813cf6b7f341ab053d89d116f407c07d5ca536f588dcb92b` to verify that a new UTXO of 580, belonging to Alice, now exist! 
Also you can verify that Alice's original UTXO has been spent and no longer exists in UtxoStore.

*Coming soon: A video walkthrough of the above demo.*

## Beginner Workshop
**Estimated time**: 2 hours

In this workshop, you will:
- Get familiar with basic Rust and Substrate functionality
- Prevent malicious users from sending bad UTXO transactions

Your challenge is to fix the code such that:
1. The Rust compiler compiles without errors
2. All tests in `utxo.rs` pass, ensuring secure transactions

### Directions
1. Checkout the `workshop` branch. The `Master` branch has the solutions, so don't peek!

```zsh
git fetch origin workshop:workshop
git checkout workshop
```

2. Cd into the base directory. Try running the test with: `cargo test -p utxo-runtime`.

```zsh
compiling utxo-runtime v2.0.0 (/Users/nicole/Desktop/utxo-workshop/runtime)
error[E0433]: failed to resolve: use of undeclared type or module `H512`
   --> /Users/nicole/Desktop/utxo-workshop/runtime/src/utxo.rs:236:31
    |
236 |             input.sigscript = H512::zero();
    |                               ^^^^ use of undeclared type or module `H512`

...
```

3. Your first task: fix all the compiler errors! Hint: Look for the `TODO` comments in `utxo.rs` to see where to fix errors.

4. Once your code compiles, it's now time to fix the `8` failing tests!

```zsh
failures:
    utxo::tests::attack_by_double_counting_input
    utxo::tests::attack_by_double_generating_output
    utxo::tests::attack_by_over_spending
    utxo::tests::attack_by_overflowing_value
    utxo::tests::attack_by_permanently_sinking_outputs
    utxo::tests::attack_with_empty_transactions
    utxo::tests::attack_with_invalid_signature
    utxo::tests::test_simple_transaction
```

5. In `utxo.rs`, edit the logic in `validate_transaction()` function to make all tests pass.

```zsh
running 8 tests
test utxo::tests::attack_by_overflowing_value ... ok
test utxo::tests::attack_by_double_counting_input ... ok
test utxo::tests::attack_by_double_generating_output ... ok
test utxo::tests::attack_by_over_spending ... ok
test utxo::tests::attack_with_empty_transactions ... ok
test utxo::tests::attack_with_invalid_signature ... ok
test utxo::tests::attack_by_permanently_sinking_outputs ... ok
test utxo::tests::test_simple_transaction ... ok
```

## Advanced Workshop
**VIDEO TUTORIALS COMING SOON**

**Estimated time**: 2 hours

In this workshop, you will implement this UTXO project from scratch using Substrate.

You will learn:
- How to implement the UTXO ledger model on Substrate
- How to secure UTXO transactions against attacks
- How to seed genesis block with UTXOs
- How to reward block validators in this environment
- How to customize transaction pool logic on Substrate
- Good coding patterns for working with Substrate & Rust, including testing and refactoring

Checkout the `startercode` branch to get the boilerplate for this workshop.
```zsh
git fetch origin startercode:startercode
git checkout startercode
```

## Helpful Resources
- [Substrate documentation](http://crates.parity.io)
- [bytes to Vec<u8> converter](https://cryptii.com/pipes/integer-encoder)
- [Polkadot UI](https://polkadot.js.org/)
