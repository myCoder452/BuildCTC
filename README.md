# Enterprise Cryptographic Tender Platform

An enterprise-grade, privacy-first procurement platform built for the **Creditcoin CC-3 EVM**. This platform leverages **Zero-Knowledge Proofs (ZK-SNARKs)** to ensure bid privacy and budget compliance while maintaining full auditability on-chain.

## 🏛️ Core Philosophy: Verifiable, Private, and Fair Procurement

The traditional procurement process is often opaque, inefficient, and susceptible to manipulation. This platform redesigns it from the ground up, prioritizing:

1.  **Cryptographic Privacy**: Bids are submitted as ZK-proofs. The actual bid value is never revealed on-chain, preventing price undercutting and information leakage. Only the proof that the bid is compliant with the tender's rules is made public.
2.  **On-Chain Auditability**: Every significant action—from tender creation to bid submission and award—is recorded as an immutable transaction on the Creditcoin blockchain, creating a tamper-proof audit trail.
3.  **Automated Compliance**: Smart contracts automatically enforce tender rules. The `BidCompliance.circom` circuit ensures that a bid is within the predefined budget range `[min, max]`, eliminating manual, error-prone checks.
4.  **Decentralized Identity**: Participants (NGOs, SMEs, Government Agents) are identified by their wallet addresses, laying the groundwork for a reputation system based on on-chain history.

---

## 🏗 System Architecture

The project is a high-performance monorepo built with **NPM workspaces**. This structure enhances modularity, simplifies dependency management, and allows for parallel development across the stack.

| Package | Port | Description |
| :--- | :--- | :--- |
| 🌐 **[frontend](packages/frontend)** | `3000` | The main enterprise dashboard for tender management and ZK-powered bidding. |
| 🔍 **[developer-explorer](packages/developer-explorer)** | `3001` | A lightweight block explorer for inspecting contract state and transaction flow. |
| ⚙️ **[backend](packages/backend)** | `3002` | The NestJS gateway for authentication, ZK-proof handling, and blockchain interaction. |
| 🏛 **[contracts](packages/contracts)** | `—` | The on-chain logic, including the `TenderRWA` and ZK `Verifier` contracts. |
| 🔐 **[circuits](packages/circuits)** | `—` | The cryptographic core, containing the `BidCompliance` ZK-SNARK circuit. |

### Detailed Package Breakdown

#### 🔐 `packages/circuits`
This is the cryptographic heart of the system, where privacy is born.

*   **Technology**: **Circom & SnarkJS**.
*   **Circuit**: `BidCompliance.circom`.
*   **Proof System**: **Groth16** over the `bn128` curve.
*   **Hashing**: **Poseidon**, a ZK-friendly hash function.
*   **Why these technologies?**:
    *   **Circom** is the industry standard for writing arithmetic circuits. Its domain-specific language (DSL) is perfectly suited for defining the complex constraints of a ZK-proof.
    *   **Groth16** is chosen for its significant advantages: it produces very small proofs and has extremely fast verification times. This is crucial for blockchain applications, as it translates directly to lower gas costs for users submitting bids.
    *   **Poseidon** is used for hashing bid data within the circuit. Unlike standard hashes like Keccak256, Poseidon is designed to be "ZK-friendly," requiring far fewer constraints in a Circom circuit, which makes the proof generation process more efficient.

#### 🏛 `packages/contracts`
The on-chain persistence and logic layer; the single source of truth.

*   **Technology**: **Solidity & Hardhat**.
*   **Main Contract**: `TenderRWA.sol` (Real-World Asset). This contract tokenizes the procurement process, managing the full lifecycle of a tender as if it were a real-world asset on the blockchain.
*   **Verifier Contract**: `Groth16Verifier.sol`. This is a highly-optimized, auto-generated contract from `snarkjs`. Its sole purpose is to verify the Groth16 proofs submitted by bidders, ensuring their bids are compliant without revealing the bid amount.
*   **Why these technologies?**:
    *   **Solidity** is the native language of the Ethereum Virtual Machine (EVM), making it the definitive choice for writing smart contracts on Creditcoin and other EVM-compatible chains.
    *   **Hardhat** provides a complete local development environment. It compiles, tests, and deploys contracts. Its ability to run a persistent local blockchain node is the cornerstone of our simplified `dev.sh` workflow, ensuring a stable and predictable state for development.

#### ⚙️ `packages/backend`
The secure enterprise gateway that connects the web to the blockchain.

*   **Technology**: **NestJS, ethers.js, JWT**.
*   **Functionality**: Provides a robust REST API for user authentication (simulated via JWT), tender management, and securely relaying ZK-proofs to the blockchain. It acts as a vital abstraction layer, simplifying frontend logic.
*   **Security**: Hardened with **Helmet** (sets secure HTTP headers) and **CORS** (restricts cross-origin requests).
*   **Why these technologies?**:
    *   **NestJS** brings a structured, modular architecture (inspired by Angular) to Node.js. For an enterprise-grade application, this is non-negotiable. It enforces a clean separation of concerns and uses Dependency Injection, which makes the codebase highly testable, maintainable, and scalable.
    *   **ethers.js** is a complete and compact library for interacting with the Ethereum blockchain. It is the modern standard for connecting a backend application to smart contracts.

#### 🌐 `packages/frontend`
The user-facing portal where all procurement activities happen.

*   **Technology**: **Next.js 14, React, ethers.js, SnarkJS**.
*   **Core Feature**: The ZK Bidding Portal. It uses `snarkjs` with WebAssembly (WASM) artifacts to generate ZK-proofs *directly in the user's browser*. This is the platform's core privacy feature, as the raw bid price never leaves the client's machine.
*   **Why these technologies?**:
    *   **Next.js** provides a world-class React framework. It enables a powerful combination of server-side rendering (for fast initial page loads) and client-side interactivity, which is essential for a responsive user experience.
    *   **Client-Side Proof Generation** is a deliberate security choice. By generating proofs in the browser, we eliminate the need to trust the server with sensitive bid data, providing maximum privacy and security to the user.

#### 🔍 `packages/developer-explorer`
A dedicated tool for transparency and debugging.

*   **Technology**: **Vite & React**.
*   **Functionality**: Provides a real-time, simplified view of blocks and transactions on the local Hardhat blockchain. It's an indispensable tool for developers to see the immediate on-chain results of their actions.
*   **Why these technologies?**:
    *   **Vite** offers a lightning-fast development server with near-instant Hot Module Replacement (HMR). For a developer-focused tool like this, a rapid and responsive development cycle is paramount, and Vite is the best-in-class solution.

---

## 🚀 Getting Started: The One-Command Launch

This project uses a "smart" startup script that automates the entire setup process.

### 1. Prerequisites
*   **Node.js**: `v18.x` or higher.
*   **NPM**: `v9.x` or higher (for monorepo workspace support).

### 2. Initial Setup
Clone the repository and install all dependencies from the root directory. NPM workspaces will automatically handle the linking of all local packages.

```bash
git clone <repository_url>
cd <repository_name>
npm install
```

### 3. Automated Workflow (Recommended)

The platform provides two primary scripts to manage the lifecycle of the environment:

#### A. Initial System Setup (`setup.sh`)
This is a one-time script that prepares the environment for the first time. It is **idempotent** and handles:
- **ZK Trusted Setup**: Generates cryptographic keys and verifier contracts.
- **State Initialization**: Cleans any stale blockchain data.
- **Mock Seeding**: Deploys contracts and seeds the network with exactly 29 blocks of mock data (5 tenders, 22 bids).

```bash
./setup.sh
```

#### B. Daily Development (`dev.sh`)
This script is used for regular development. It ensures the environment is always ready by:
- **Port Management**: Terminates any orphaned node processes.
- **Node Sync**: Automatically re-seeds the blockchain if the node starts in a fresh/empty state.
- **Service Orchestration**: Parallelly launches the Frontend, Backend, and Developer Explorer.

```bash
./dev.sh
```

### 4. Application Access
Once active, the platform is accessible at:
- **Dashboard**: [http://localhost:3000](http://localhost:3000)
- **Explorer**: [http://localhost:3001](http://localhost:3001)
- **Backend API**: [http://localhost:3002](http://localhost:3002)

---

## 🛠 Manual Setup Guide

For developers who require granular control or wish to debug specific components:

### 1. ZK Circuit Compilation
Navigate to the circuits package and initialize the cryptographic primitives.
```bash
cd packages/circuits
npm install
npm run compile
bash setup_zk.sh
```

### 2. Smart Contract Deployment
Start the local Hardhat node and deploy the TenderRWA ecosystem.
```bash
# Terminal 1
cd packages/contracts
npx hardhat node

# Terminal 2
cd packages/contracts
npx hardhat deploy --network localhost
npx hardhat seed --network localhost
```

### 3. Starting Platform Services
Services can be started individually using NPM workspaces from the project root:
```bash
# Start the Backend (Port 3002)
npm run start:dev --workspace=@procurement/backend

# Start the Frontend (Port 3000)
npm run dev --workspace=@procurement/frontend

# Start the Explorer (Port 3001)
npm run dev --workspace=developer-explorer -- --port 3001
```

---

## 🛡 Security & Design Principles

*   **Privacy by Default**: The system is architected to minimize data exposure. The core principle is that no sensitive information (like a bid price) should be revealed unless cryptographically necessary.
*   **Client-Side Proof Generation**: ZK-proofs are generated in the browser. This is a critical security measure. The user's raw bid data never touches the backend server, mitigating the risk of server-side compromises.
*   **Immutability as a Feature**: The blockchain's immutability is leveraged to create a permanent, unchangeable record of all procurement activities, ensuring accountability for all participants.
*   **Gas Efficiency**: The `Groth16Verifier.sol` contract is highly optimized for gas, as it is the most frequently called on-chain component. The Groth16 proof system is chosen for its small proof size and efficient verification.
