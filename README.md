# Rusty-Bitcoin: A Toy Blockchain Protocol in Rust

Rusty-Bitcoin is a project to build a simplified, Bitcoin-like blockchain from scratch using Rust. The primary goal is to gain a deep, first-principles understanding of the core components that make a cryptocurrency protocol work.

This project deliberately avoids a complex smart contract VM (like the EVM) and instead focuses on the foundational technologies of Bitcoin:

*   **Public/Private Key Cryptography** for digital identity and ownership.
*   **The UTXO (Unspent Transaction Output) Model** for state management.
*   **Merkle Trees** for transaction integrity and efficiency.
*   **Proof-of-Work (PoW)** for consensus.
*   **A P2P Network** for decentralized communication and consensus.

---

## Technical Architecture

The system is a command-line application that runs a node. Each node exposes a JSON-RPC API for interaction and communicates with other nodes to maintain a single, consistent ledger.

### Core Components

1.  **Cryptography (`crypto` module)**
    *   **Wallet (`wallet.rs`):** Manages the generation of `secp256k1` keypairs (private and public keys) using the `k256` crate. The public key serves as the user's identity.
    *   **Merkle Tree (`merkle.rs`):** A function that takes a list of transactions and computes a single `merkle_root` hash. This root is included in the block header to securely and efficiently summarize all transactions within the block.

2.  **The UTXO Model (`transaction` module)**
    *   This project implements the Unspent Transaction Output model, which is fundamentally different from a simple account balance model.
    *   **`TxOutput` (`utxo.rs`):** Represents a "coin" of a certain `value` that is "locked" to a specific `public_key_hash`. Only the holder of the corresponding private key can unlock and spend it.
    *   **`TxInput` (`utxo.rs`):** Represents a reference to a previous `TxOutput`. It contains a signature to prove that the spender has the right to consume the output.
    *   **`Transaction` (`tx.rs`):** A data structure that consumes one or more existing UTXOs (as inputs) and creates one or more new UTXOs (as outputs). Transactions must be signed by the spender to be valid. The total value of inputs must be greater than or equal to the total value of outputs.

3.  **The Blockchain (`blockchain` module)**
    *   **`Block` (`block.rs`):** A container for a set of transactions, linked to the previous block. Its header includes a timestamp, nonce, previous block's hash, and the Merkle root of its transactions.
    *   **`Blockchain` (`chain.rs`):** Manages the list of blocks (`Vec<Block>`), the mempool of pending transactions, and the current UTXO set. It contains the logic for Proof-of-Work mining and validating new blocks.

4.  **The Node (`node.rs` and `api.rs`)**
    *   **P2P Consensus:** The node maintains a list of peers. It implements a `resolve_conflicts` method that follows the "longest valid chain wins" rule. When a conflict occurs, the node queries its peers, finds the longest valid chain, and adopts it. This process involves rebuilding its local UTXO set by replaying the transactions from the new, authoritative chain.
    *   **API:** An `axum`-based web server exposes endpoints for interacting with the node (e.g., creating a wallet, checking a balance, submitting a transaction, and triggering mining).

---

## Project Phases (60-Hour Plan)

This project is broken down into four distinct, 15-hour phases.

### Phase 1: Foundational Structures & Merkle Trees (Hours 1-15)
*   **Goal:** Create the immutable chain structure and implement transaction hashing.
*   **Tasks:**
    1.  Setup project with `cargo new rusty-bitcoin`.
    2.  Define `Block` and initial `Transaction` structs in their respective modules.
    3.  Implement the `build_merkle_root` function in `crypto::merkle`.
    4.  Modify `Block` to include the `merkle_root`.
    5.  Implement a basic Proof-of-Work mining loop.

### Phase 2: Cryptography & Wallets (Hours 16-30)
*   **Goal:** Implement digital identities and transaction signing.
*   **Tasks:**
    1.  Add `k256` and `ecdsa` to `Cargo.toml`.
    2.  Implement the `Wallet` struct and key generation in `crypto::wallet`.
    3.  Define the final structures for `TxInput` and `TxOutput` in `transaction::utxo`.
    4.  Implement `sign_transaction` and `verify_transaction` methods on the `Transaction` struct.

### Phase 3: The UTXO Ledger Model (Hours 31-45)
*   **Goal:** Implement Bitcoin's core state management model.
*   **Tasks:**
    1.  Replace any temporary state model in `Blockchain` with a `UTXOSet`. A `HashMap` is a good starting point.
    2.  Implement the comprehensive transaction validation logic:
        *   Verify that inputs point to unspent outputs.
        *   Verify the cryptographic signature for each input.
        *   Ensure input value >= output value.
    3.  Modify the `add_block` function to correctly update the `UTXOSet` (removing spent outputs, adding new ones).

### Phase 4: API & P2P Networking (Hours 46-60)
*   **Goal:** Expose the protocol's functionality and enable decentralization.
*   **Tasks:**
    1.  Set up `axum` and `tokio`. Wrap the main node state in an `Arc<Mutex<T>>`.
    2.  Implement API endpoints: `/wallet/new`, `/balance/:address`, `/transactions`, `/mine`.
    3.  Implement P2P logic in `node.rs`:
        *   Add peer registration endpoint `/nodes/register`.
        *   Implement `resolve_conflicts` method.
        *   Add `/nodes/resolve` endpoint to trigger consensus.
    4.  Test the full system by running two nodes and having them sync.

---

## How to Run the Project

1.  **Install Rust:** If you haven't already, [install Rust](https://www.rust-lang.org/tools/install).
2.  **Clone & Build**:
    ```bash
    git clone <your-repo-url>
    cd rusty-bitcoin
    cargo build --release
    ```
3.  **Run a Node**:
    Run a single node on port 3000.
    ```bash
    cargo run -- --port 3000
    ```
4.  **Run a Second Node**:
    Open a new terminal and run a peer node on a different port.
    ```bash
    cargo run -- --port 3001
    ```

### Example API Usage (`curl`)

*   **Register Node 2 as a peer of Node 1:**
    ```bash
    curl -X POST -H "Content-Type: application/json" -d '{"nodes": ["http://127.0.0.1:3001"]}' http://localhost:3000/nodes/register
    ```
*   **Mine a block on Node 1 (this will contain the coinbase transaction):**
    ```bash
    curl http://localhost:3000/mine
    ```
*   **Tell Node 2 to sync with its peers (it will discover Node 1's new block):**
    ```bash
    curl http://localhost:3000/nodes/resolve
    ```
*   **Check the chains on both nodes to confirm they match:**
    ```bash
    curl http://localhost:3000/blocks
    curl http://localhost:3001/blocks
    ```
