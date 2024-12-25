---
sidebar_position: 4
---

# Place a Limit Order

Now that your BalanceManager is funded, you can start placing limit orders in a DeepBook pool. This guide walks you through the process of creating and submitting a limit order using the DeepBook Rust SDK.

---

## What is a Limit Order?

A **Limit Order** allows you to specify:
- The **price** at which you want to buy or sell.
- The **quantity** of tokens to trade.
- Whether the order is a **bid** (buy) or an **ask** (sell).

Limit orders remain in the order book until they are matched or canceled.

---

## Step-by-Step Guide to Place a Limit Order

Follow these steps to place a limit order in a DeepBook pool:

:::tip Full Source Code
For the full example, visit the [GitHub example](https://github.com/styu12/deepbook-rust-sdk/blob/main/examples/place_limit_order.rs).
:::

### 1. Initialize the Balance Manager

First, configure your environment and initialize `BalaceManagerMap` with the required BalanceManager details you want to place the limit order for.

```rust
use deepbook::client::DeepBookClient;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Step 1: Initialize Sui client for writing
    let (sui, sender, _receiver) = utils::setup_for_write().await?;

    // Step 2: Define environment
    let env = "testnet";

    // Step 3: Initialize balance managers
    let mut balance_managers: BalanceManagerMap = HashMap::new();
    balance_managers.insert(
        "MANAGER_1".to_string(),
        BalanceManager {
            address: <your BalanceManager Object ID>.to_string(),
            // use trade_cap if you're not the owner of the balance manager
            trade_cap: None,
        },
    );

    // Step 3: Place a limit order
    place_order(&client, "DEEP_SUI", "MANAGER_1", "123456789", 0.02, 10.0, true).await?;

    Ok(())
}
```

### 2. Initialize the DeepBook Client

Next, initialize the `DeepBookClient` with the `DeepBookConfig` object.
```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    ...
    // Step 4: Initialize DeepBookClient with DeepBookConfig
    let db_config = DeepBookConfig::new(
        env,
        sender.to_string(),
        None,
        Some(balance_managers),
        None,
        None,
    );
    let db_client = DeepBookClient::new(Arc::new(sui.clone()), Arc::new(db_config));
    ...
}   
```

### 3. Place the Limit Order

Here’s how to define and place your limit order:

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    ...
    // Step 5: Add place_limit_order transaction to PTB with deepbook-sdk
    let mut ptb = ProgrammableTransactionBuilder::new();
    match db_client.deep_book.place_limit_order(
        &mut ptb,
        "DEEP_SUI", // pool_id
        "MANAGER_1", // balance_manager_key
        "123456789", // client_order_id
        0.02, // price
        10.0, // amount
        true, // is_bid
        None,
        None,
        None,
        None,
    ).await {
        Ok(_) => println!("add place_limit_order transaction to PTB"),
        Err(e) => {
            println!("Error placing limit order: {}", e);
            for source in e.chain() {
                println!("Caused by: {}", source);
            }
        },
    }

    // Step 6: Execute the transaction block
    if let Err(e) = execute_transaction_block(&sui, ptb, sender).await {
        println!("Error executing transaction block for 'place_limit_order'");
        for source in e.chain() {
            println!("Caused by: {}", source);
        }
    }

    Ok(())
    ...
}   
```

### 4. Execute the Transaction Block

The function `execute_transaction_block` submits the transaction to the Sui blockchain and confirms its execution.
It is same as [the one used in the Create a Balance Manager guide](./create-a-balance-manager.md#3-execute-the-transaction-block).

### Example Output

After successfully placing a limit order, you should see a transaction response similar to this:
```shell
------------------------------------
Transaction Results
[hash]
 "2o2mKC9TCseFY3UU4ruiufApHJw41L5iuaCFzTU4tLUj"

[effect]
 Some(V1(SuiTransactionBlockEffectsV1 { status: Success, executed_epoch: 594, gas_used: GasCostSummary { computation_cost: 1000000, storage_cost: 50251200, storage_rebate: 48763044, non_refundable_storage_fee: 492556 }, modified_at_versions: [SuiTransactionBlockEffectsModifiedAtVersions { object_id: 0x0c0b8562949131dce849495894848be477510114f707cd6cb9bd2a7354cf7c14, sequence_number: SequenceNumber(302109375) }, SuiTransactionBlockEffectsModifiedAtVersions { object_id: 0x0cb45faadd6c3769bd825dfd3538e34d6c658a0b55a8caa52e03c46b07aef8b9, sequence_number: SequenceNumber(296857706) }, SuiTransactionBlockEffectsModifiedAtVersions { object_id: 0x0d1b1746d220bd5ebac5231c7685480a16f1c707a46306095a4c67dc7ce4dcae, sequence_number: SequenceNumber(302109376) }, SuiTransactionBlockEffectsModifiedAtVersions { object_id: 0x1f70c07921cb873a95ad2010fe6b53b37d0830b657288aa0c3df6686ff937361, sequence_number: SequenceNumber(296857706) }, SuiTransactionBlockEffectsModifiedAtVersions { object_id: 0x9310d9ec3ad998152a770f3baf4a0289c5832dc4df77419665fac353c5480dfc, sequence_number: SequenceNumber(296857706) }, SuiTransactionBlockEffectsModifiedAtVersions { object_id: 0xaa4b9c70ba93e6a0d174686e639a6ca703fd6ffdd24d804829ed5fabe5a68bdf, sequence_number: SequenceNumber(302109375) }, SuiTransactionBlockEffectsModifiedAtVersions { object_id: 0xf14907baa0db6165eb74ac10e72afad15494a8249fe7356c83891772b90306c4, sequence_number: SequenceNumber(296857706) }], shared_objects: [SuiObjectRef { object_id: 0x0cb45faadd6c3769bd825dfd3538e34d6c658a0b55a8caa52e03c46b07aef8b9, version: SequenceNumber(296857706), digest: o#EQ4WPoUfuy4YLXcvPeFjCbR9jAGxgRcdq6bjbQZhE6TG }, SuiObjectRef { object_id: 0x0d1b1746d220bd5ebac5231c7685480a16f1c707a46306095a4c67dc7ce4dcae, version: SequenceNumber(302109376), digest: o#8DAnykUbTZY2b1NqQ9t2p9yss4gx6Jr1HrR98oHXZRfE }, SuiObjectRef { object_id: 0x0000000000000000000000000000000000000000000000000000000000000006, version: SequenceNumber(262528576), digest: o#4rdhH5fTur63t2UcUHP4A47n8ogEHFkPDJEHUTno6jT5 }], transaction_digest: TransactionDigest(2o2mKC9TCseFY3UU4ruiufApHJw41L5iuaCFzTU4tLUj), created: [], mutated: [OwnedObjectRef { owner: ObjectOwner(0x8b10b885724f3caeb6911a14170f1814fecd783284302eba0add6419dfff9f89), reference: SuiObjectRef { object_id: 0x0c0b8562949131dce849495894848be477510114f707cd6cb9bd2a7354cf7c14, version: SequenceNumber(302109377), digest: o#4uC4YxCeNQ2LJZFSg17PSKywXVBnJ2tvhbinsoFYJnAR } }, OwnedObjectRef { owner: Shared { initial_shared_version: SequenceNumber(47602850) }, reference: SuiObjectRef { object_id: 0x0cb45faadd6c3769bd825dfd3538e34d6c658a0b55a8caa52e03c46b07aef8b9, version: SequenceNumber(302109377), digest: o#2K3ZNRNQ6pCirwFJuDonD1VR5aMxLDPpdMu7Gjotmxu6 } }, OwnedObjectRef { owner: Shared { initial_shared_version: SequenceNumber(159250500) }, reference: SuiObjectRef { object_id: 0x0d1b1746d220bd5ebac5231c7685480a16f1c707a46306095a4c67dc7ce4dcae, version: SequenceNumber(302109377), digest: o#7Jz61YY7iRgA1bN7gJUQSfFCzTwiD4xpJmiGANA7Dapw } }, OwnedObjectRef { owner: AddressOwner(0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e), reference: SuiObjectRef { object_id: 0x1f70c07921cb873a95ad2010fe6b53b37d0830b657288aa0c3df6686ff937361, version: SequenceNumber(302109377), digest: o#92RUFRqbcxrkPM5ETpbTscf8a3u16ZcTrNeDuARjJ3xw } }, OwnedObjectRef { owner: ObjectOwner(0x6768095bef0b39ac5f373358e784d967d90127c6c0bc061a7338a24909c0a0db), reference: SuiObjectRef { object_id: 0x9310d9ec3ad998152a770f3baf4a0289c5832dc4df77419665fac353c5480dfc, version: SequenceNumber(302109377), digest: o#Pg6GiKnFtv2NoR7AMVmfF1KmCrBCHiHJasSG7G5oV8m } }, OwnedObjectRef { owner: ObjectOwner(0x0b07315b25ae29979e33f959fcf0e4bdc2ae34ee0600496ad642b96b7a76901e), reference: SuiObjectRef { object_id: 0xaa4b9c70ba93e6a0d174686e639a6ca703fd6ffdd24d804829ed5fabe5a68bdf, version: SequenceNumber(302109377), digest: o#H5FepqCvhW62Yt4NHsjDyn5iyWWtifwaCRYjJtWFB29Y } }, OwnedObjectRef { owner: ObjectOwner(0x04696c4b1d28ff84dcc325de3f38685ee06d60e9ef1d3212c7459d7c76b65c09), reference: SuiObjectRef { object_id: 0xf14907baa0db6165eb74ac10e72afad15494a8249fe7356c83891772b90306c4, version: SequenceNumber(302109377), digest: o#BP4dwfvGBwvsjtWt1iiFSLUYcLdxjtdVmfrPf5WLcXca } }], unwrapped: [], deleted: [], unwrapped_then_deleted: [], wrapped: [], gas_object: OwnedObjectRef { owner: AddressOwner(0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e), reference: SuiObjectRef { object_id: 0x1f70c07921cb873a95ad2010fe6b53b37d0830b657288aa0c3df6686ff937361, version: SequenceNumber(302109377), digest: o#92RUFRqbcxrkPM5ETpbTscf8a3u16ZcTrNeDuARjJ3xw } }, events_digest: Some(TransactionEventsDigest(AtHVyVSuDXb7WsTTZXKotVsMGhr2RSUwXL1z4U5J7eBU)), dependencies: [TransactionDigest(47cD2WNP9HSBdQSUy834WjqcvsEqmPE2Ys7FVz76qvQH), TransactionDigest(4xjtwxou3TJRx2dZgiLbCZbzh4W1kVgTUeUaKtRXH6Ja), TransactionDigest(A6xFwPXThpVqgKmHxffCXbVmanE9kcK2CX5LrqAMXwBD), TransactionDigest(C4qArTRhhEUGWEdvVB8v9UkpSdVuqpbrcoKmfHdCTu7b), TransactionDigest(Gtwgse64nSVXhQvmqCpwCe5xJz9N4VypvEGJUy5DyG4e), TransactionDigest(HWuUiwr7rpDsS1WC7v8Gnq4esfK6QuhsF1H4yLSK5ZFN)] }))

[object changes]:
 Some([Mutated { sender: 0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e, owner: ObjectOwner(0x8b10b885724f3caeb6911a14170f1814fecd783284302eba0add6419dfff9f89), object_type: StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("dynamic_field"), name: Identifier("Field"), type_params: [U64, Struct(StructTag { address: a706f5eebcfde58ee1b03d642f222c646fe09b8d2d7a59fbee0fdc73fa21eb33, module: Identifier("big_vector"), name: Identifier("Slice"), type_params: [Struct(StructTag { address: a706f5eebcfde58ee1b03d642f222c646fe09b8d2d7a59fbee0fdc73fa21eb33, module: Identifier("order"), name: Identifier("Order"), type_params: [] })] })] }, object_id: 0x0c0b8562949131dce849495894848be477510114f707cd6cb9bd2a7354cf7c14, version: SequenceNumber(302109377), previous_version: SequenceNumber(302109375), digest: o#4uC4YxCeNQ2LJZFSg17PSKywXVBnJ2tvhbinsoFYJnAR }, Mutated { sender: 0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e, owner: Shared { initial_shared_version: SequenceNumber(47602850) }, object_type: StructTag { address: a706f5eebcfde58ee1b03d642f222c646fe09b8d2d7a59fbee0fdc73fa21eb33, module: Identifier("balance_manager"), name: Identifier("BalanceManager"), type_params: [] }, object_id: 0x0cb45faadd6c3769bd825dfd3538e34d6c658a0b55a8caa52e03c46b07aef8b9, version: SequenceNumber(302109377), previous_version: SequenceNumber(296857706), digest: o#2K3ZNRNQ6pCirwFJuDonD1VR5aMxLDPpdMu7Gjotmxu6 }, Mutated { sender: 0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e, owner: Shared { initial_shared_version: SequenceNumber(159250500) }, object_type: StructTag { address: a706f5eebcfde58ee1b03d642f222c646fe09b8d2d7a59fbee0fdc73fa21eb33, module: Identifier("pool"), name: Identifier("Pool"), type_params: [Struct(StructTag { address: 36dbef866a1d62bf7328989a10fb2f07d769f4ee587c0de4a0a256e57e0a58a8, module: Identifier("deep"), name: Identifier("DEEP"), type_params: [] }), Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("sui"), name: Identifier("SUI"), type_params: [] })] }, object_id: 0x0d1b1746d220bd5ebac5231c7685480a16f1c707a46306095a4c67dc7ce4dcae, version: SequenceNumber(302109377), previous_version: SequenceNumber(302109376), digest: o#7Jz61YY7iRgA1bN7gJUQSfFCzTwiD4xpJmiGANA7Dapw }, Mutated { sender: 0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e, owner: AddressOwner(0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e), object_type: StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("coin"), name: Identifier("Coin"), type_params: [Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("sui"), name: Identifier("SUI"), type_params: [] })] }, object_id: 0x1f70c07921cb873a95ad2010fe6b53b37d0830b657288aa0c3df6686ff937361, version: SequenceNumber(302109377), previous_version: SequenceNumber(296857706), digest: o#92RUFRqbcxrkPM5ETpbTscf8a3u16ZcTrNeDuARjJ3xw }, Mutated { sender: 0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e, owner: ObjectOwner(0x6768095bef0b39ac5f373358e784d967d90127c6c0bc061a7338a24909c0a0db), object_type: StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("dynamic_field"), name: Identifier("Field"), type_params: [Struct(StructTag { address: a706f5eebcfde58ee1b03d642f222c646fe09b8d2d7a59fbee0fdc73fa21eb33, module: Identifier("balance_manager"), name: Identifier("BalanceKey"), type_params: [Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("sui"), name: Identifier("SUI"), type_params: [] })] }), Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("balance"), name: Identifier("Balance"), type_params: [Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("sui"), name: Identifier("SUI"), type_params: [] })] })] }, object_id: 0x9310d9ec3ad998152a770f3baf4a0289c5832dc4df77419665fac353c5480dfc, version: SequenceNumber(302109377), previous_version: SequenceNumber(296857706), digest: o#Pg6GiKnFtv2NoR7AMVmfF1KmCrBCHiHJasSG7G5oV8m }, Mutated { sender: 0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e, owner: ObjectOwner(0x0b07315b25ae29979e33f959fcf0e4bdc2ae34ee0600496ad642b96b7a76901e), object_type: StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("dynamic_field"), name: Identifier("Field"), type_params: [U64, Struct(StructTag { address: a706f5eebcfde58ee1b03d642f222c646fe09b8d2d7a59fbee0fdc73fa21eb33, module: Identifier("pool"), name: Identifier("PoolInner"), type_params: [Struct(StructTag { address: 36dbef866a1d62bf7328989a10fb2f07d769f4ee587c0de4a0a256e57e0a58a8, module: Identifier("deep"), name: Identifier("DEEP"), type_params: [] }), Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("sui"), name: Identifier("SUI"), type_params: [] })] })] }, object_id: 0xaa4b9c70ba93e6a0d174686e639a6ca703fd6ffdd24d804829ed5fabe5a68bdf, version: SequenceNumber(302109377), previous_version: SequenceNumber(302109375), digest: o#H5FepqCvhW62Yt4NHsjDyn5iyWWtifwaCRYjJtWFB29Y }, Mutated { sender: 0x63caf24ab6dfc41c44fd67e7e117e2f0a4ef0f636fed9ddddde2f1bd230bae8e, owner: ObjectOwner(0x04696c4b1d28ff84dcc325de3f38685ee06d60e9ef1d3212c7459d7c76b65c09), object_type: StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("dynamic_field"), name: Identifier("Field"), type_params: [Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("object"), name: Identifier("ID"), type_params: [] }), Struct(StructTag { address: a706f5eebcfde58ee1b03d642f222c646fe09b8d2d7a59fbee0fdc73fa21eb33, module: Identifier("account"), name: Identifier("Account"), type_params: [] })] }, object_id: 0xf14907baa0db6165eb74ac10e72afad15494a8249fe7356c83891772b90306c4, version: SequenceNumber(302109377), previous_version: SequenceNumber(296857706), digest: o#BP4dwfvGBwvsjtWt1iiFSLUYcLdxjtdVmfrPf5WLcXca }])


Process finished with exit code 0
```

### Recap
- Limit orders let you specify price, quantity, and bid/ask options for trading. 
- Use the place_limit_order method to submit your order programmatically. 
- Verify the transaction success in the output logs.

In the next guide, learn how to Fetch Open Orders and manage your trading activity efficiently.