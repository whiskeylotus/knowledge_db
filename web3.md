# Web3 / Blockchain

## Keywords 

DeFi        Decentralized Finance — protocols like Aave, Uniswap.
DApp        Decentralized Application.
CEX/DEX     Centralized / Decentralized Exchange.
Custodian   Entity that holds crypto assets securely.
RPC Endpoint    Interface to interact with blockchain nodes.
Wallet Exploit / Seed Leak  Attacks that compromise private keys or mnemonics.

## Blockchain Basics

| Concept | Explanation | Example / Tip |
|----------|--------------|----------------|
| **Blockchain** | A decentralized, append-only ledger maintained by a network of nodes. | Bitcoin, Ethereum |
| **Block** | Contains transactions + hash of previous block (ensures immutability). | Changing data breaks the chain. |
| **Consensus Mechanism** | Ensures all nodes agree on a single state of the ledger. | PoW (Bitcoin), PoS (Ethereum) |
| **Node** | A computer participating in maintaining the blockchain. | Full node stores complete blockchain. |
| **Transaction** | Signed data that triggers a state change on the blockchain. | Sending ETH or calling a smart contract. |
| **Gas** | Fee paid for computation on the blockchain. Prevents spam and infinite loops. | `gasLimit`, `gasPrice` in transactions. |
| **Nonce** | Counter ensuring each transaction from an account is unique. | Prevents replay attacks. |

---

## Smart Contracts (Core Concepts)

| Concept | Explanation | Example |
|----------|--------------|----------|
| **Smart Contract** | Self-executing code deployed on a blockchain that runs when triggered by transactions. | A Solidity contract managing a token sale. |
| **EVM (Ethereum Virtual Machine)** | Runtime that executes smart contract bytecode deterministically. | All EVM chains (Polygon, BSC, Avalanche). |
| **Solidity** | Primary language for Ethereum smart contracts. | Syntax similar to JavaScript/C++. |
| **ABI (Application Binary Interface)** | Describes contract’s functions and types for external interactions. | Used by `web3.js` or `ethers.js`. |
| **State Variables** | Stored permanently on-chain; form contract’s state. | `uint public balance; address owner;` |
| **Events** | Emit logs accessible off-chain (for UI or monitoring). | `emit Transfer(msg.sender, to, amount);` |
| **Modifiers** | Reusable access control logic. | `modifier onlyOwner { require(msg.sender == owner); _; }` |
| **Fallback / Receive** | Triggered when contract receives ETH without matching function. | Payment handling. |
| **Require / Revert** | Halt execution and revert state if condition fails. | `require(amount > 0, "Invalid amount");` |
