# EIP-8003: Epistemic Context Lattice (ECL) Standard
## A Standard for Verifiable, Compressed AI Context on Ethereum
### Status: Draft
### Type: Standards Track - ERC
### Category: ERC
### Author: @hyperkit-dev
### Created: 2026-02-07
### Requires: ERC-721, ERC-1155, EIP-4337, EIP-712, ERC-8004

# 📋 Abstract
This proposal defines a standard protocol for representing, compressing, and verifying Large Language Model (LLM) context windows as on-chain attestations. The Epistemic Context Lattice (ECL) enables AI agents to maintain cryptographically provable, infinitely extensible memory while reducing storage costs by 1000x through hyperdimensional computing and Merkleized differential state.

## The standard specifies:
- ERC-8003-Context: Interface for compressed context NFTs
- ERC-8003-Graph: Schema for knowledge graph attestations
- ERC-8003-Verifier: On-chain verification of context integrity

## Core Concepts
| Component         | Solidity Interface | Purpose                                    |
| ----------------- | ------------------ | ------------------------------------------ |
| Context Crystal   | IERC8003Crystal    | Hyperdimensional vector storage (10K dims) |
| Epistemic Graph   | IERC8003Graph      | Semantic relationship attestation          |
| Merkle Context    | IERC8003Merkle     | Differential state verification            |
| Memory Controller | IERC8003Controller | On-chain memory economics                  |

## Who Its for
| Persona             | Use Case                          | Implementation                          |
| ------------------- | --------------------------------- | --------------------------------------- |
| AI Agent Developers | Build agents with infinite memory | Import @erc8003/sdk                     |
| DeFi Protocols      | Verifiable oracle context         | Integrate ERC8003Verifier contract      |
| DAO Operators       | Transparent governance AI         | Deploy ECLGovernanceAdapter             |
| Data Curators       | Sell high-quality context graphs  | Mint context as ERC-1155 semi-fungibles |
| Model Providers     | Prove training data provenance    | Anchor dataset hashes via ECL           |
- Layer 2s: Optimism/Arbitrum for cheap graph updates
- Storage Networks: IPFS/Arweave for hypervector blobs
- ZK Provers: RISC Zero/Axiom for private similarity proofs
- AI Frameworks: LangChain/LlamaIndex plugins

# Example Flow
```mermaid
graph TD;
    User_Agent["User/AI Agent"]
    SDK["ERC8003 SDK"]
    Ethereum_L2["Ethereum L2 \n(Graph contract)"]
    Ethereum_L1["Ethereum L1 \n(Crystal + Verifier contracts)"]
    Storage["Storage \n(IPFS/Arweave blobs)"]

    User_Agent -->|User Query| SDK
    SDK -->|Compress| Ethereum_L2
    Ethereum_L2 -->|Bind| Ethereum_L1
    Ethereum_L1 -->|Store| Storage
    Storage -->|Retrieve| SDK
```
# 1. Core Architecture (Layered Stack)
```mermaid
flowchart TB
    %% User + Agent Layer
    subgraph L0[User / Agent Layer]
        U[User]
        A[AI Agent\nLangChain / ElizaOS]
        U --> A
    end

    %% SDK Layer
    subgraph L1[erc8003/sdk Layer]
        C[Compress\nText → Hypervector]
        D[Decompress\nHypervector → Text]
        A --> C
        D --> A
    end

    %% Chain + Storage
    subgraph L2[Ethereum + Storage Layer]
        subgraph L2a[Ethereum L1]
            CR["IERC8003Crystal\ncrystallize()"]
            VR["IERC8003Verifier\nverifyRelevance()"]
        end
        subgraph L2b[Ethereum L2]
            GR["IERC8003Graph\nbind()/getRelevantSubgraph()"]
        end
        subgraph L2c[Storage]
            ST["IPFS / Arweave\nHypervector blobs"]
        end
    end

    %% Flows
    C --> ST
    ST --> CR
    CR --> GR
    A --> GR
    GR --> D
    A --> VR

    classDef sdk fill:#8B5CF6,stroke:#4C1D95,color:#ffffff
    classDef chain fill:#627EEA,stroke:#3730A3,color:#ffffff
    classDef storage fill:#059669,stroke:#047857,color:#ffffff

    class C,D sdk
    class CR,VR,GR chain
    class ST storage
```

# 2. Sequence Diagram: Full Write/Read Flow (Temporal Interactions)
```mermaid
sequenceDiagram
    autonumber
    participant User
    participant Agent
    participant SDK as ECL SDK
    participant IPFS as IPFS/Arweave
    participant L1 as L1 Crystal
    participant L2 as L2 Graph
    participant Ver as Verifier

    %% Write (Crystallize)
    User->>Agent: Ask question / new info
    Agent->>SDK: sendText()
    SDK->>SDK: compress(text) → hypervector
    SDK->>IPFS: store(hypervector)
    IPFS-->>SDK: cid
    SDK->>L1: crystallize(hypervector, proof)
    L1-->>SDK: contextId
    SDK->>L2: bind(prevContextId, contextId, TEMPORAL)

    %% Read (Query)
    User->>Agent: "Remind me about liquidation risk"
    Agent->>SDK: queryContext(query)
    SDK->>SDK: compress(query) → queryVector
    SDK->>L2: getRelevantSubgraph(queryVector)
    L2-->>SDK: contextIds[]
    SDK->>IPFS: fetchBlobs(contextIds)
    IPFS-->>SDK: hypervectors[]
    SDK->>SDK: decompress(hypervectors) → snippets
    SDK->>Ver: verifyRelevance(contextId, queryVector)
    Ver-->>SDK: ok, similarity score
    SDK-->>Agent: contextSnippets
    Agent-->>User: answer with cited context

```
# 3. Flowchart: Compression Pipeline (Process Flow)
```mermaid
flowchart TD
    T[Raw Text<br/>N tokens] --> HDC[Hyperdimensional Encoding<br/>VSA bind/bundle]
    HDC --> Q[Quantize to 2 bits/dim<br/>10k dims]
    Q --> P[Pack into uint256 156<br/>1.25 KB]
    P --> H[SHA3-256<br/>contentHash]
    H --> M[Build Merkle Tree<br/>merkleRoot]
    P --> B[Store Blob<br/>IPFS / Arweave CID]
    M --> Ctx[crystallize - contextId on L1]

    %% Optional reverse path
    Ctx -.-> RQ[queryRelevant -] -.-> D[Decompress<br/>Hypervector → approx text]

    classDef sdk fill:#8B5CF6,stroke:#4C1D95,color:#ffffff
    classDef chain fill:#627EEA,stroke:#3730A3,color:#ffffff
    classDef storage fill:#059669,stroke:#047857,color:#ffffff

    class HDC,Q,P,H,M sdk
    class B storage
    class Ctx chain
    class D sdk
```
# 4. Agent Integration Example (LangChain Flow)
```mermaid
flowchart LR
    subgraph Client[Client App]
        U[User]
        LC[LangChain Agent]
    end

    subgraph Mem[Memory Backend]
        EM[ERC8003Memory - custom BaseMemory]
        SDK[erc8003_sdk]
    end

    subgraph Chain[Ethereum + Storage]
        L1C[IERC8003Crystal]
        L2G[IERC8003Graph]
        ST[IPFS / Arweave]
    end

    %% Interactions
    U --> LC
    LC --> EM:::memNode

    EM --> SDK
    SDK --> L1C
    SDK --> L2G
    SDK --> ST

    L1C --> SDK
    L2G --> SDK
    ST --> SDK

    SDK --> EM
    EM --> LC
    LC --> U

    classDef memNode fill:#F97316,stroke:#C2410C,color:#ffffff
    classDef chain fill:#627EEA,stroke:#3730A3,color:#ffffff
    classDef storage fill:#059669,stroke:#047857,color:#ffffff
    classDef sdk fill:#8B5CF6,stroke:#4C1D95,color:#ffffff

    class EM memNode
    class SDK sdk
    class L1C,L2G chain
    class ST storage
```
# 5. Security Layers (Defense‑in‑Depth)
```mermaid
flowchart TD
    IN[User / External Input] --> SAN[Perception Layer<br/>Sanitization + Filters]
    SAN --> MEM[Context Layer<br/>Per-user Signed Memory]
    MEM --> GUARD[Guardian Model<br/>Memory Injection Detector]
    GUARD --> ECL[ERC-8003 Layer<br/>crystallize/query]
    ECL --> ACT[Decision Layer<br/>Action Whitelist + Spend Limits]
    ACT --> HITL[Human-in-the-Loop<br/>High-value review]
    HITL --> EXEC[Action Layer<br/>On-chain Execution]

    %% Notes
    %% SAN: Prompt detectors, OWASP filters, Rate limiting
    %% MEM: Per-user buckets, TTL on entries, ECDSA signatures
    %% ECL: Hypervector compression, Merkle roots on L1, Graph on L2
    %% ACT: Allowed contracts, Daily spend caps, Circuit breakers

    classDef sec fill:#F59E0B,stroke:#B45309,color:#ffffff
    classDef ecl fill:#627EEA,stroke:#3730A3,color:#ffffff

    class SAN,MEM,GUARD,ACT,HITL,EXEC sec
    class ECL ecl
```

## [Read more click this]()
