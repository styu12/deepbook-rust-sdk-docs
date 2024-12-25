---
sidebar_position: 2
---

# Deposit Funds into a Balance Manager

To interact with DeepBook and place orders, your **BalanceManager Object** must have sufficient funds. Tokens such as **SUI**, **DEEP**, or **DBUSDC** should be deposited into your BalanceManager beforehand.

:::warning Take care
If you haven’t created a BalanceManager yet, please follow the [Create a Balance Manager guide](./create-a-balance-manager.md) first.
:::

---

## Why Deposit Funds?

Depositing funds into a BalanceManager is essential for:
- **Order Placement**: You cannot place buy or sell orders without sufficient balance in your BalanceManager.

---

## Step-by-Step Guide to Deposit Funds

Below is a guide to deposit funds into a BalanceManager Object using the DeepBook Rust SDK.

:::tip Full Source Code
For the full example, check out the [source code on GitHub](https://github.com/styu12/deepbook-rust-sdk/blob/main/examples/deposit_into_manager.rs).
:::

### 1. Initialize the Balance Manager

First, configure your environment and initialize `BalaceManagerMap` with the required BalanceManager details you want to deposit funds into.

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
            trade_cap: None,
        },
    );
    ...
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

### 3. Deposit Funds into the Manager

The following code demonstrates how to deposit funds into a BalanceManager:

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    ...
    // Step 5: Add deposit_into_manager transaction to PTB with deepbook-sdk
    let mut ptb = ProgrammableTransactionBuilder::new();
    match db_client.balance_manager.deposit_into_manager(
        &mut ptb,
        "MANAGER_1",
        "SUI",
        0.1,
    ).await {
        Ok(_) => println!("add deposit transaction to PTB (0.1 SUI for MANAGER_1)"),
        Err(e) => {
            println!("Error depositing into MANAGER_1");
            for source in e.chain() {
                println!("Caused by: {}", source);
            }
        },
    }

    // Step 6: Execute the transaction block
    if let Err(e) = execute_transaction_block(&sui, ptb, sender).await {
        println!("Error executing transaction block for 'deposit_into_manager'");
        for source in e.chain() {
            println!("Caused by: {}", source);
        }
    }

    Ok(())
}   
```

### 4. Execute the Transaction Block

The function `execute_transaction_block` submits the transaction to the Sui blockchain and confirms its execution.
It is same as [the one used in the Create a Balance Manager guide](./create-a-balance-manager.md#3-execute-the-transaction-block).


### Recap
- Ensure your BalanceManager has sufficient funds before placing orders.
- Follow the guide above to deposit tokens like SUI, DEEP, or DBUSDC. 

To verify your deposit, you’ll need to check your BalanceManager’s balance. learn how to do this in the [next guide](./check-manager-balance.md).