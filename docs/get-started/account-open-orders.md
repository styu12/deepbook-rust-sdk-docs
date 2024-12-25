---
sidebar_position: 5
---

# Fetch Open Orders

After placing limit orders, you may want to track them. This guide explains how to fetch all open orders for a specific BalanceManager in a DeepBook pool using the DeepBook Rust SDK.

---

## Why Fetch Open Orders?

- **Monitor Active Orders**: View all orders that are yet to be fulfilled or canceled.
- **Manage Trading Activity**: Decide whether to modify or cancel open orders based on market conditions.
- **Ensure Transparency**: Validate that your orders have been correctly recorded in the order book.

---

## Step-by-Step Guide to Fetch Open Orders

Follow these steps to query open orders for a BalanceManager:

:::tip Full Source Code
For the full example, visit the [GitHub example](https://github.com/styu12/deepbook-rust-sdk/blob/main/examples/account_open_orders.rs).
:::

### 1. Initialize the Balance Manager

First, configure your environment and initialize `BalaceManagerMap` with the required BalanceManager details you want to check open orders for.

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
            address: "0x0cb45faadd6c3769bd825dfd3538e34d6c658a0b55a8caa52e03c46b07aef8b9".to_string(),
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

### 3. Fetch Open Orders

The following code demonstrates how to query open orders for a specific pool and BalanceManager:

- pools: A list of DeepBook pools you want to fetch open orders for. (e.g., ["SUI_DBUSDC", "DEEP_SUI"])
- manager: The identifier of the BalanceManager (e.g., “MANAGER_1”).

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    ...
    // Step 5: Define the pool and manager which you want to fetch open orders for
    let pools = vec!["SUI_DBUSDC", "DEEP_SUI"];
    let manager = "MANAGER_1";

    // Step 6: Call account_open_orders with deepbook-sdk and check the response
    println!("------------------------------------");
    println!("Sui RPC Response");
    for pool in pools {
        match db_client.account_open_orders(pool, manager).await {
            Ok(orders) => {
                println!("[pool]\n {:?}", pool);
                println!("[orders]\n {:?}\n", orders);
            },
            Err(e) => println!("Error fetching orders for pool {}: {}", pool, e),
        }
    }
    println!("------------------------------------");

    Ok(())
    ...
}   
```

### Example Output

A successful query will return a list of open order IDs for the specified BalanceManager:

```shell
------------------------------------
Sui RPC Response
[pool]
 "SUI_DBUSDC"
[orders]
 []

[pool]
 "DEEP_SUI"
[orders]
 [368934881492637776393709403713, 368934881492637776393709403090, 368934881492637776393709403086]

------------------------------------

```

### Recap
- Use the account_open_orders method to fetch active orders for a specific BalanceManager and pool. 
- Open orders can help you monitor and manage your trading strategy effectively.

### Congratulations on Completing the Basics!

You’ve successfully completed the Get Started guide for the DeepBook Rust SDK. 🎉
From creating a BalanceManager to placing and monitoring orders, you now have the foundational knowledge to interact with the DeepBook protocol effectively.

### What’s Next?

Ready to take your skills to the next level?
Explore advanced features and best practices to enhance your trading strategies:
- Understand DeepBook’s Architecture: Learn how the protocol handles matching, settlement, and performance optimization. 
- Dive into Advanced Examples: Explore topics like governance voting, flash loans, and administrative tasks. 
- Join the Community: Collaborate with other developers in the Sui ecosystem and share your experiences.

Thank you for choosing the *DeepBook Rust SDK*. We’re excited to see what you’ll build. 🚀
Check out the full documentation or visit our [GitHub repository](https://github.com/styu12/deepbook-rust-sdk) to stay updated!