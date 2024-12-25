---
sidebar_position: 1
---

# Create a Balance Manager

A **BalanceManager Object** is essential for interacting with the DeepBook protocol. It serves as the primary entity for managing funds, placing orders, and interacting with pools on the Sui blockchain. If you already have a BalanceManager Object, you can skip this step. Otherwise, follow the instructions below to create a new one.

---

## Why Do You Need a Balance Manager?

The BalanceManager Object acts as a shared account for trading activities:
- **Order Placement**: Required for placing and managing limit orders in pools.
- **Funds Management**: Centralized management of token balances for seamless transactions.
- **Protocol Compatibility**: Essential for interacting with DeepBook’s smart contracts.

Without a BalanceManager Object, you cannot utilize DeepBook’s features.

---

## Step-by-Step Guide to Create a Balance Manager

Here’s how to create a BalanceManager Object programmatically using the DeepBook Rust SDK:

:::tip Full Source Code
Want to see the complete example? Check out the [full source code on GitHub](https://github.com/styu12/deepbook-rust-sdk/blob/main/examples/create_and_share_balance_manager.rs).
::: 

### 1. Initialize the DeepBook Client
First, configure your environment and initialize the `DeepBookClient`. Make sure to set up the Sui client and DeepBook configuration correctly.

```rust
use deepbook::client::DeepBookClient;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Step 1: Initialize Sui client for writing
    let (sui, sender, receiver) = utils::setup_for_write().await?;

    // Step 2: Define environment
    let env = "testnet";

    // Step 3: Initialize DeepBookClient with DeepBookConfig
    let db_config = DeepBookConfig::new(
        env,
        sender.to_string(),
        None,
        None,
        None,
        None,
    );
    let db_client = DeepBookClient::new(Arc::new(sui.clone()), Arc::new(db_config));
    ...
}
```

### 2. Create and Share a BalanceManager

The following code demonstrates how to create and share a new BalanceManager:

```rust
{
    ...
     // Step 4: Add create_and_share_balance_manager transaction to PTB with deepbook-sdk
    let mut ptb = ProgrammableTransactionBuilder::new();
    match db_client.balance_manager.create_and_share_balance_manager(
        &mut ptb,
    ) {
        Ok(_) => println!("add create_and_share_balance_manager transaction to PTB"),
        Err(e) => {
            println!("Error creating and sharing balance manager");
            for source in e.chain() {
                println!("Caused by: {}", source);
            }
        },
    }
    
    // Step 5: Execute the transaction block
    if let Err(e) = execute_transaction_block(&sui, ptb, sender).await {
        println!("Error executing transaction block for 'create_and_share_balance_manager'");
        for source in e.chain() {
            println!("Caused by: {}", source);
        }
    }

    Ok(())
}
```

### 3. Execute the Transaction Block

The execute_transaction_block function submits the transaction to the Sui blockchain and confirms its execution:

```rust
/// Execute a transaction block with the given programmable transaction builder and sender address.
/// Transaction will be signed based on your local Sui Keystore Configuration. (located at ~/.sui/sui_config/sui.keystore)
pub async fn execute_transaction_block(
    client: &SuiClient,
    ptb: ProgrammableTransactionBuilder,
    sender: SuiAddress,
) -> Result<()> {
    println!("Building the transaction...");
    let pt = ptb.finish();

    // get coins for gas fee
    let coins = get_all_coins(client, sender, SUI_COIN_TYPE).await
        .map_err(|e| anyhow::anyhow!("Failed to get coins for gas fee: {e}"))?;

    // build the transaction data
    let gas_price = client.read_api().get_reference_gas_price().await?;
    let tx_data = TransactionData::new_programmable(
        sender,
        coins
            .iter()
            .map(|coin| coin.object_ref())
            .collect::<Vec<_>>(),
        pt,
        10_000_000, // gas_budget (0.01 SUI)
        gas_price,
    );

    // sign transaction
    let keystore = FileBasedKeystore::new(&sui_config_dir()?.join(SUI_KEYSTORE_FILENAME))?;
    let signature = keystore.sign_secure(&sender, &tx_data, Intent::sui_transaction())?;

    // execute the transaction
    println!("Executing the transaction...");
    let transaction_response = client
        .quorum_driver_api()
        .execute_transaction_block(
            Transaction::from_data(tx_data, vec![signature]),
            SuiTransactionBlockResponseOptions::full_content(),
            Some(ExecuteTransactionRequestType::WaitForLocalExecution),
        )
        .await?;

    // print transaction results
    println!("------------------------------------");
    println!("Transaction Results");
    println!("[hash]\n {:?}\n", transaction_response.digest.to_string());
    println!("[effect]\n {:?}\n", transaction_response.effects);
    println!("[object changes]:\n {:?}\n", transaction_response.object_changes);

    Ok(())
}
```

## Recap

- If you already have a BalanceManager Object, you can skip this step.
- not, follow the above guide to create and share a new BalanceManager Object.
- the DeepBook SDK to programmatically manage your trading accounts efficiently.

Ready to move on? Learn how to Deposit Funds into a Manager in the next section.