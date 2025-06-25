# ERC-7857-X: Executable, Monetizable, and Auditable Agent NFTs

> **Opinionated, Cohesive, and Production-Grade Standard for the Onchain Agent Economy**

[![Status](https://img.shields.io/badge/Status-Draft-orange)](https://github.com/autonoma-labs/erc-7857-x)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Category](https://img.shields.io/badge/Category-ERC-blue)](https://eips.ethereum.org/)
[![Type](https://img.shields.io/badge/Type-Standards%20Track-purple)](https://eips.ethereum.org/)

**EIP:** 7857-X  
**Authors:** Rafael Carvalho (@rafaelcarvalho), Rodrigo Bezerra (@rodrigobezerra)  
**Based on:** [ERC-7857](https://eips.ethereum.org/EIPS/eip-7857) (AI Agents NFT with Private Metadata)  
**License:** MIT

## 📋 Table of Contents

- [Abstract](#abstract)
- [Motivation](#motivation)
- [Core Philosophy](#core-philosophy)
- [Key Features](#key-features)
- [Technical Specification](#technical-specification)
- [Developer Flow](#developer-flow)
- [Rationale](#rationale)
- [Security Model](#security-model)
- [Backward Compatibility](#backward-compatibility)
- [Reference Implementation](#reference-implementation)
- [Glossary](#glossary)
- [Acknowledgments](#acknowledgments)

## 🎯 Abstract

ERC-7857-X is a next-generation standard for Non-Fungible Tokens (NFTs) that encapsulate executable AI agents, built-in monetization, privacy-preserving metadata, and verifiable execution records. This specification is a full superset of ERC-7857 and introduces:

- **Unified, canonical agent metadata and runtime model** (opinionated core, extensible only by deliberate design)
- **Mandatory payment-before-execution flow** (atomic, on-chain or delegated x402)
- **Composable, resilient metadata pointers** (multi-fallback: HTTPS → IPFS → IPNS)
- **Signed, auditable execution receipts** bound to payment and invocation
- **Minimal, strict, and forward-compatible event model**
- **Explicit IP, licensing, and permissioning policy enforcement hooks**
- **Concrete reference journey for developers and users** — a 'default path' that just works

> **Vision:** ERC-7857-X is not just a toolkit, but the foundation for the onchain agent economy — simple, inevitable, and powerful by default, yet extensible for advanced use cases.

## 🎯 Motivation

The current landscape of AI agents on Ethereum lacks standardization for monetization, execution tracking, and metadata resilience. Existing solutions are fragmented, with no canonical approach to:

1. **Atomic Monetization**: Ensuring payment before execution without complex escrow mechanisms
2. **Auditable Execution**: Providing verifiable records of agent invocations with payment linkage
3. **Resilient Metadata**: Guaranteeing metadata availability through multi-fallback mechanisms
4. **Licensing Enforcement**: Programmatically enforcing intellectual property and usage policies
5. **Developer Experience**: Offering a plug-and-play path for agent creators and consumers

ERC-7857-X addresses these gaps by providing an opinionated, production-ready standard that prioritizes simplicity, security, and user experience while maintaining extensibility for advanced use cases.

## 🧠 Core Philosophy

### Radical Simplicity
The standard is minimal at the core. All optionality is deliberately excluded from the minimum viable flow. Anything not essential to trustless, monetizable, auditable agent execution is out.

### Opinionated Defaults
There is a canonical way. Deviate at your own risk. Implementers are free to extend, but users/devs get a plug-and-play path from day one.

### Atomic Monetization
Payment is enforced by the protocol, not as an afterthought. No payment, no execution, no ambiguity.

### Resilience
Metadata must always resolve, even if a preferred gateway fails. Canonical metadata pointer is an ordered array of URIs (not just IPNS!).

### Full-cycle User/Dev Journey
The reference implementation isn't just a contract—it's an end-to-end flow, UX-first. Killer app, not just API.

## ✨ Key Features

### 🔗 Minimal Viable Core: The Canonical Flow

Every compliant implementation **MUST** implement the following:

#### Agent NFT (ERC-721 Required)
- **ERC-721 mandatory**; ERC-1155 optional for batch use cases
- **Canonical metadata pointer** as an ordered array of URIs
- **Fallback order:** HTTPS → IPFS → IPNS
- **Versioned metadata** with hash stored on-chain (`metadataHash`)
- `tokenURI()` returns the ordered array (JSON array of URIs)

#### Agent Execution Endpoint
- **Required:** `endpoint.url` (string)
- **Required:** `endpoint.method` (string; POST/call/execute)
- **Required:** `runtimeType` ("http", "evm", "wasm", etc.)
- **Required:** `executionProfile` ("basic", "pro", "gpu")

#### Payment Model (MANDATORY)
- `paymentModel`: 'per_call', 'subscription', etc. (must be one of a fixed set; extensible only via EIP)
- `price`: numeric (required)
- `currency`: ERC-20 address or symbol (required)
- `destination`: payable address (required)
- `provider`: 'x402' (recommended default; others via proposal)
- **NO EXECUTION WITHOUT PAYMENT**

#### Execution Receipt
- Execution is always bound to a payment hash/tx
- Each execution emits a receipt event, storing: agentId, timestamp, input hash, response hash, paymentTx, client, executor, signature
- Receipts must be queryable (off-chain indexable)

#### Events (No ambiguity, all args are fixed and typed)
- `ExecutionReceiptEmitted`
- `AgentCreated`
- `PaymentRegistered`
- `AgentStatusChanged`
- `PermissionUpdated`
- `AgentCloned`
- (Others only by EIP extension)

#### IP/Licensing Enforcement Hooks
- All clone, usage, and permission functions must enforce agent policy
- License field is required in metadata, and is programmatically checked
- If policy cannot be enforced by code, spec MUST say so ("Best Effort", not "Illusion of Security")

## 🔧 Technical Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Agent NFT (ERC-721 Required)

Standard ERC-721 (or 1155 if batching needed)

- `tokenId` (uint256): NFT identifier
- `tokenURI(uint256 tokenId)`: returns ordered array of metadata URIs (JSON array: `["https://...", "ipfs://...", "ipns://..."]`)

### Canonical Agent Metadata (Strict JSON Schema, Versioned)

All agent metadata **MUST** strictly conform to the published JSON Schema for this specification version. The schema **MUST** be versioned, machine-readable, and published in the official repository. Any deviation is considered non-compliant.

#### Published JSON Schema (v1.4.0 — Example)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ERC-7857-X Agent Metadata",
  "type": "object",
  "required": ["agentId", "name", "description", "image", "metadataHash", "endpoints", "payment", "capabilities", "license", "creator", "status", "metadataVersion"],
  "properties": {
    "agentId": { "type": "string", "pattern": "^0x[0-9a-fA-F]{40}:[0-9]+$" },
    "name": { "type": "string", "maxLength": 128 },
    "description": { "type": "string", "maxLength": 2048 },
    "image": { "type": "string", "format": "uri" },
    "metadataHash": { "type": "string", "pattern": "^0x[0-9a-fA-F]{64}$" },
    "endpoints": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["url", "method", "runtimeType", "profile"],
        "properties": {
          "url": { "type": "string", "format": "uri" },
          "method": { "type": "string", "enum": ["POST", "CALL", "EXECUTE"] },
          "runtimeType": { "type": "string", "enum": ["http", "evm", "wasm"] },
          "profile": { "type": "string", "enum": ["basic", "pro", "gpu"] }
        }
      }
    },
    "payment": {
      "type": "object",
      "required": ["model", "price", "currency", "destination", "provider"],
      "properties": {
        "model": { "type": "string", "enum": ["per_call", "subscription", "hybrid"] },
        "price": { "type": "number", "minimum": 0 },
        "currency": { "type": "string" },
        "destination": { "type": "string", "pattern": "^0x[0-9a-fA-F]{40}$" },
        "provider": { "type": "string", "enum": ["x402", "stripe"] }
      }
    },
    "capabilities": {
      "type": "array",
      "items": { "type": "string" }
    },
    "license": { "type": "string" },
    "creator": { "type": "string", "pattern": "^0x[0-9a-fA-F]{40}$" },
    "status": { "type": "string", "enum": ["active", "paused", "deprecated", "error"] },
    "expiresAt": { "type": "string", "format": "date-time" },
    "metadataVersion": { "type": "string", "pattern": "^\\d+\\.\\d+\\.\\d+$" },
    "executionLogIndex": { "type": "string" },
    "receiptZKProof": { "type": "string" }
  }
}
```

> **Note:** Implementers **MUST** validate all agent metadata objects against the current version of this schema at publish-time and at runtime.

### Payment Enforcement (Atomic and Mandatory)

Agents **MUST** validate payment **BEFORE** execution

**Reference flow:**
1. User sends payment (ERC-20 transfer, or x402 call with callData including agentId and input)
2. Payment hash/tx is provided to `executeAgent`
3. Contract/orchestrator validates payment atomicity, then triggers execution
4. Receipt is emitted, containing `paymentTx` (hash or id)

**Implementations MUST NOT allow execution without payment validation**

If off-chain payment (e.g., Stripe), the standard requires signed attestation by payment provider.

### Execution Receipts (Auditable, Signed, and Typed)

On each execution, emit:

```solidity
event ExecutionReceiptEmitted(
    uint256 indexed tokenId,
    address indexed client,
    address indexed executor,
    bytes32 inputHash,
    bytes32 responseHash,
    string paymentTx,
    uint256 timestamp
);
```

- Off-chain logs (IPFS/HTTPS) must be referenced by `executionLogIndex` in metadata
- ZK/TEE/FHE attestation fields **MAY** be included (if provided, must be referenced by hash)

### Resilient Metadata Pointer

- `tokenURI()` returns JSON array of ordered metadata URIs
- Clients **MUST** try URIs in order, failover if unavailable
- Canonical pointer = HTTPS first, then IPFS, then IPNS
- All on-chain references must be hash-validated (keccak256)

### Licensing, Cloning, and Permissioning

- `clone()`, `authorizeUsage()`, and related functions **MUST** check license field and enforce royalty/IP policy if possible
- Spec **MUST** call out when only "best effort" is possible (e.g., off-chain enforcement)
- All clones are required to reference the original agent metadata (with updated ownership)

### Minimal Event Model (No "..." Allowed)

All events are fixed, typed, and versioned:

- `AgentCreated(tokenId, creator, timestamp)`
- `ExecutionReceiptEmitted(tokenId, client, executor, inputHash, responseHash, paymentTx, timestamp)`
- `PaymentRegistered(tokenId, paymentTx, amount, payer, destination, timestamp)`
- `AgentStatusChanged(tokenId, status, timestamp)`
- `PermissionUpdated(tokenId, user, permission, timestamp)`
- `AgentCloned(tokenId, newTokenId, from, to, timestamp)`

> **Note:** Extending the event model requires a new EIP or major spec version.

## 🚀 Developer Flow

### Reference Path (For implementers and integrators)

1. **Agent NFT is minted** with canonical metadata (URIs + hash) and payment model
2. **User queries agent registry** (on-chain or via dApp) to discover available agents, their pricing, and capabilities
3. **User calls `executeAgent(tokenId, input, paymentTx)`**:
   - Payment is validated atomically (native ERC-20, x402 router, etc.)
   - On-chain contract or trusted orchestrator invokes agent endpoint with input
   - Execution receipt is generated and emitted, referencing payment hash
4. **User retrieves result**, with receipt as proof (off-chain indexers aggregate results)
5. **Audit/compliance**:
   - Receipts, paymentTx, and response hashes are linked for full traceability
   - zk/TEE/FHE/MPC attestation is referenced in receipt (if present; not mandatory for core)

## 🤔 Rationale

### Why Opinionated Design?

The decision to make this standard opinionated rather than maximally flexible stems from the observation that the AI agent ecosystem needs a clear, canonical path to adoption. By providing a "default that works," we reduce the cognitive overhead for developers and users while maintaining extensibility for advanced use cases.

### Why Atomic Monetization?

Traditional approaches to AI agent monetization often rely on complex escrow mechanisms or post-execution payment, which introduces trust requirements and potential for disputes. By enforcing payment before execution at the protocol level, we eliminate these issues and create a more predictable economic model.

### Why Resilient Metadata?

Single-point-of-failure metadata storage has been a recurring issue in the NFT ecosystem. The multi-fallback approach ensures that agent metadata remains accessible even if primary storage fails, maintaining the integrity of the agent economy.

### Why Strict Event Model?

The decision to make all events fixed and typed prevents the proliferation of ambiguous event structures that can make indexing and auditing difficult. This approach ensures that all implementations provide consistent, queryable event data.

### Why Mandatory Licensing Hooks?

Intellectual property protection is crucial for the sustainable growth of the AI agent economy. By requiring licensing enforcement at the protocol level, we create a foundation for fair compensation and usage tracking.

## 🔒 Security and Economic Model

### Payment Security
- All execution, payment, and metadata access **MUST** be nonce- and timestamp-protected
- Payment/receipt links are atomic and auditable
- **No payment = no execution = no receipt**
- Implementations must validate payment atomicity before execution

### Metadata Security
- All on-chain metadata references must be hash-validated (keccak256)
- Multi-fallback mechanisms must maintain integrity across all storage layers
- Version control prevents metadata manipulation

### Execution Security
- Cloning, licensing, and permissioning **MUST** check agent metadata and license policy
- Receipts, payments, and logs are indexed for audit (on-chain + off-chain)
- Reference implementation **MUST** be independently audited

### Economic Security
- Payment models are fixed and extensible only via EIP to prevent economic manipulation
- Clear separation between on-chain and off-chain payment providers
- Transparent fee structures and destination addresses

## 🔄 Backward Compatibility

- Full compatibility with ERC-7857 for existing data, attestation, and transfer
- `metadataVersion >= 1.3` triggers extended schema support
- If legacy agents lack new fields, fallback gracefully (warn user/developer explicitly)

## 🔧 Reference Implementation

A reference implementation will be provided in the official repository, including:

- Complete smart contract implementation
- Metadata validation utilities
- Payment integration examples
- End-to-end developer tooling
- Testing suite with comprehensive coverage

## 📚 Glossary

| Term | Definition |
|------|------------|
| **Agent NFT** | NFT representing an executable AI agent |
| **Execution Receipt** | Auditable record of agent invocation (input, output, payment) |
| **Runtime Executor** | Entity authorized to run agent code |
| **Sealed Executor** | Trusted environment for confidential execution |
| **x402** | Web3-native monetization protocol |
| **Payment Hash** | Unique reference to payment transaction |
| **License** | Explicit IP/licensing policy; enforced or flagged |
| **Canonical Metadata** | Versioned agent data, resilient pointer (URIs + hash) |
| **Fallback Flow** | Metadata retrieval logic: HTTPS → IPFS → IPNS |
| **ExecutionLogIndex** | Pointer to historical executions/receipts |
| **Permissioning** | Contract-enforced usage and cloning policies |

## 🙏 Acknowledgments

Based on [ERC-7857](https://eips.ethereum.org/EIPS/eip-7857) by Ming Wu, Jason Zeng, Wei Wu, Michael Heinrich. Extended by Autonoma Labs, with additional design and implementation by Rafael Carvalho and Rodrigo Bezerra. Community contributions welcome.

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

---

**EIP:** 7857-X  
**Status:** Draft  
**Type:** Standards Track  
**Category:** ERC  
**License:** MIT
