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
This is the cryptographic heart of the system.

*   **Technology**: **Circom & SnarkJS**.
*   **Circuit**: `BidCompliance.circom`.
*   **Proof System**: **Groth16** over the `bn128` curve.
*   **Functionality**: It takes a private bid price and a public budget range (`min`, `max`) as inputs. It outputs a `proof` and `publicSignals` (the Poseidon hash of the bid and the budget range). The proof confirms the bid is within the budget without revealing the price.
*   **Why Circom?**: Circom is the industry standard for writing arithmetic circuits. Its DSL is expressive and specifically designed for ZK-proofs, and its ecosystem (SnarkJS) provides the tools for in-browser proof generation and verification.

#### 🏛 `packages/contracts`
The on-chain persistence and logic layer.

*   **Technology**: **Solidity & Hardhat**.
*   **Main Contract**: `TenderRWA.sol` (Real-World Asset). This contract manages the full lifecycle of a tender, from creation to awarding and fulfillment. It integrates the ZK verifier to validate bids.
*   **Verifier Contract**: `Verifier.sol`. This is an auto-generated contract produced by `snarkjs`. Its sole purpose is to verify the Groth16 proofs submitted by bidders. It is highly gas-optimized.
*   **Why Hardhat?**: Hardhat provides a robust, extensible, and fast environment for local development, testing, and deployment of Solidity smart contracts. Its integration with `ethers.js` and its rich plugin ecosystem make it ideal for a complex project like this.

#### ⚙️ `packages/backend`
The secure enterprise gateway.

*   **Technology**: **NestJS, ethers.js, JWT**.
*   **Authentication**: A mock JWT-based system (`accounts.txt`) simulates a real-world user database, assigning roles (NGO, SME) to wallet addresses.
*   **Tender Module**: Provides REST endpoints for creating tenders, fetching tender data, and submitting ZK-signed bids. It acts as a trusted intermediary that validates proofs before broadcasting transactions to the Creditcoin network.
*   **Security**: Hardened with **Helmet** (sets secure HTTP headers), **CORS** (restricts cross-origin requests), and follows standard API security best practices.
*   **Why NestJS?**: NestJS brings a structured, modular architecture to Node.js, heavily inspired by Angular. This is critical for enterprise applications, as it enforces a clean separation of concerns (modules, controllers, services) and simplifies maintenance and scalability.

#### 🌐 `packages/frontend`
The user-facing portal for all procurement activities.

*   **Technology**: **Next.js 14, React, ethers.js, SnarkJS**.
*   **ZK Bidding Portal**: The most critical feature. It uses `snarkjs` with WASM artifacts (generated from the circuit) to create ZK-proofs *directly in the user's browser*. This is paramount for privacy, as the raw bid price never leaves the client machine.
*   **UI/UX**: A premium, enterprise-grade dark-navy theme with glassmorphism effects. It provides real-time status tracking of tenders and on-chain activities.
*   **Why Next.js?**: Next.js offers the best of both worlds: the power of React for building dynamic UIs, combined with server-side rendering (SSR) and static site generation (SSG) for performance. Its file-based routing and API routes are perfect for building a full-featured web application.

#### 🔍 `packages/developer-explorer`
A dedicated tool for transparency and debugging.

*   **Technology**: **Vite & React**.
*   **Functionality**: Provides a real-time view of blocks and transactions on the connected blockchain (e.g., a local Hardhat node). It allows developers to inspect the internal state of the `TenderRWA` and `Verifier` contracts.
*   **Why Vite?**: For a developer tool, speed is key. Vite offers a lightning-fast development experience with near-instant hot module replacement (HMR), making it the ideal choice for this lightweight explorer.

---

## 🚀 Deployment & Operational Guide

### 1. Prerequisites
*   **Node.js**: `v18.x` or higher.
*   **NPM**: `v9.x` or higher (for workspace support).
*   **Circom**: *(Optional)* Required only if you intend to modify and recompile the ZK circuits. Install via `npm install -g circom`.

### 2. Initial Setup
Clone the repository and install all dependencies from the root directory. NPM workspaces will automatically link the packages.

```bash
git clone <repository_url>
cd cryptographic-tender-monorepo
npm install
```

### 3. ZK Circuit Compilation & Trusted Setup
This step generates the cryptographic artifacts required for proof generation and verification. It only needs to be run once, or whenever the circuit logic in `packages/circuits/BidCompliance.circom` is changed.

```bash
cd packages/circuits

# 1. Compile the circuit
# This generates the R1CS constraint system and the WASM witness calculator.
npm run compile

# 2. Perform the Trusted Setup
# This generates the proving key, verification key, and the Verifier.sol contract.
# It uses the "Powers of Tau" ceremony for security.
bash setup_zk.sh
```
**Note**: The `setup_zk.sh` script automatically copies the generated `Verifier.sol` into the `packages/contracts` directory, ensuring the on-chain component is always in sync with the circuit.

### 4. Smart Contract Deployment
Deploy the contracts to a local network or a public testnet.

```bash
cd packages/contracts

# 1. Start a local blockchain node
# This provides a sandboxed EVM environment for testing.
npx hardhat node

# 2. Deploy the contracts (in a separate terminal)
# This script deploys TenderRWA.sol and the Verifier.sol.
npx hardhat run scripts/deploy_zk.ts --network localhost
```
After deployment, contract addresses will be printed to the console. These need to be copied into the backend and frontend configuration for proper connectivity.

### 5. Manual Execution Guide

For more granular control, you can run each service manually. Open a new terminal for each command.

**Terminal 1: Start the Local Blockchain**

This command starts a local, in-memory blockchain instance using Hardhat, which is essential for development and testing.

```bash
# Navigate to the contracts package
cd packages/contracts

# Start the Hardhat node
npx hardhat node
```

**Terminal 2: Deploy Smart Contracts**

Once the node is running, deploy the `TenderRWA` and `Verifier` contracts to the local network.

```bash
# Navigate to the contracts package
cd packages/contracts

# Deploy the contracts
npx hardhat run scripts/deploy_zk.ts --network localhost
```

**Terminal 3: Start the Backend Server**

The NestJS backend acts as the API gateway, connecting the frontend to the blockchain.

```bash
# From the project root
npm run start:backend
```

**Terminal 4: Start the Frontend Application**

This command launches the Next.js user dashboard.

```bash
# From the project root
npm run start:frontend
```

**Terminal 5: Start the Developer Explorer**

This lightweight block explorer allows you to monitor on-chain activity in real-time.

```bash
# From the project root
npm run start:explorer
```

Once all services are running, you can access the applications at their respective ports:
*   **Main Dashboard**: `http://localhost:3000`
*   **Developer Explorer**: `http://localhost:3001`
*   **Backend API**: `http://localhost:3002`

---

## 🛡 Security & Design Principles

*   **Privacy by Default**: The system is architected to minimize data exposure. The core principle is that no sensitive information (like a bid price) should be revealed unless cryptographically necessary.
*   **Client-Side Proof Generation**: ZK-proofs are generated in the browser. This is a critical security measure. The user's raw bid data never touches the backend server, mitigating the risk of server-side compromises.
*   **Immutability as a Feature**: The blockchain's immutability is leveraged to create a permanent, unchangeable record of all procurement activities, ensuring accountability for all participants.
*   **Gas Efficiency**: The `Verifier.sol` contract is highly optimized for gas, as it is the most frequently called on-chain component. The Groth16 proof system is chosen for its small proof size and efficient verification.
