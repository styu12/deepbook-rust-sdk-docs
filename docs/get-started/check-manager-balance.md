---
sidebar_position: 3
---

# Check Manager Balance

After depositing funds into your **BalanceManager Object**, the next step is to verify the deposit. This guide explains how to check the balance of your BalanceManager using the DeepBook Rust SDK.

---

## Why Check the Balance?

- **Verify Deposits**: Ensure that your deposited tokens (e.g., SUI, DEEP, DBUSDC) have been successfully credited.
- **Track Spending**: Monitor your available balance to manage trades effectively.
- **Prevent Errors**: Avoid failed transactions by checking for sufficient funds beforehand.

---

## Step-by-Step Guide to Check Manager Balance

Follow these steps to fetch and verify your BalanceManager's balance:

:::tip Full Source Code
For the full source code, visit the [GitHub example](https://github.com/styu12/deepbook-rust-sdk/blob/main/examples/check_manager_balance.rs).
:::

### 1. Initialize the DeepBook Client

First, configure your environment and initialize `BalaceManagerMap` with the required BalanceManager details you want to check the balance for.

```rust
use deepbook::client::DeepBookClient;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Step 1: Initialize Sui client
    let (sui, sender) = setup_for_read().await?;

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

### 3. Check the Balance

The following code demonstrates how to query the balance of a specific BalanceManager for a given token:
- manager_key: The identifier of the BalanceManager (e.g., `MANAGER_1`).
- coin_key: The token you want to query (e.g., `SUI`, `DEEP`).

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Step 5: Call check_manager_balance with deepbook-sdk and check the response
    println!("------------------------------------");
    println!("Sui RPC Response");
    match db_client.check_manager_balance("MANAGER_1", "SUI").await {
        Ok(balance) => {
            println!("[manager balance]\n {:?}\n", balance);
        },
        Err(e) => {
            println!("Error fetching balance");
            for source in e.chain() {
                println!("Caused by: {}", source);
            }
        },
    }
    println!("------------------------------------");

    Ok(())
}
```

### Example Output

When the balance query is successful, you should see output similar to the following:

```shell
------------------------------------
Sui RPC Response
[manager balance]
 Object {"coin_type": String("0x0000000000000000000000000000000000000000000000000000000000000002::sui::SUI"), "balance": Number(2.0)}

------------------------------------

```

### Recap
- Use the check_manager_balance method to verify the tokens in your BalanceManager.
- This step ensures your deposited funds are available for trading and other operations.

Next, learn how to Place a Limit Order using your funded BalanceManager.
