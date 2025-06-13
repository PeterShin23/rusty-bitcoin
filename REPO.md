```
rusty-bitcoin/
├── .gitignore
├── Cargo.toml
├── README.md
└── src/
    ├── main.rs         // Entry point, CLI parsing, starts the node and API.
    ├── api.rs          // All web server logic (Axum/Actix-web routes).
    ├── node.rs         // The main Node struct, P2P logic, and consensus.
    │
    ├── blockchain/
    │   ├── mod.rs      // Declares the blockchain sub-modules.
    │   ├── block.rs    // Definition of the `Block` struct and its implementation.
    │   └── chain.rs    // Manages the chain itself, mining, and adding new blocks.
    │
    ├── crypto/
    │   ├── mod.rs      // Declares the crypto sub-modules.
    │   ├── wallet.rs   // Wallet generation (private/public keys).
    │   └── merkle.rs   // Merkle Tree generation logic.
    │
    └── transaction/
        ├── mod.rs      // Declares the transaction sub-modules.
        ├── tx.rs       // The main `Transaction` struct and its signing/verification logic.
        └── utxo.rs     // Defines `TxInput`, `TxOutput`, and logic for the UTXO set.
```
