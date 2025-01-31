# Quantum-Resistant Blockchain: Consensus Mechanism

## **Purpose**
The consensus mechanism ensures agreement among nodes on the blockchain’s state. 
It replaces classical algorithms (e.g., ECDSA, SR25519) with **CRYSTALS-Dilithium**, a post-quantum signature scheme, 
to secure block production and validation against quantum attacks.

---

## **Why CRYSTALS-Dilithium?**
### **Algorithm Overview**
- **Type**: Lattice-based digital signature scheme.
- **Security Foundation**: Relies on the hardness of two mathematical problems:
    1. **Learning With Errors (LWE)**
    2. **Short Integer Solution (SIS)**
- **NIST Standardization**: Part of the NIST Post-Quantum Cryptography Project’s **Round 3 finalists** (Category: Digital Signatures).

### **Quantum Resistance**
- **Shor’s Algorithm Immunity**: LWE/SIS problems are **not efficiently solvable** by quantum computers, unlike integer 
- factorization (RSA) or discrete logarithms (ECC).
- **Long-Term Security**: Estimated security levels:
    - **Dilithium2**: 128-bit security (NIST Level 1).
    - **Dilithium3**: 192-bit security (NIST Level 3).
    - **Dilithium5**: 256-bit security (NIST Level 5).

---

## **Workflow: Block Signing and Validation**
### **Step 1: Block Proposal**
1. **Leader Election**:
    - Validators are selected via **Pure PoS** based on staked tokens and quantum-resistant randomness (SHA-3-based VRFs).
2. **Block Creation**:
    - The elected validator packages transactions into a block and computes a **SHA-3 hash** of the block content.

### **Step 2: Block Signing**
1. **Private Key Usage**:
    - The validator signs the block header (including the SHA-3 hash) using their **private Dilithium key**.
2. **Signature Generation**:
    - Dilithium generates a lattice-based signature using polynomial ring operations.

### **Step 3: Block Validation**
1. **Public Key Verification**:
    - Nodes verify the signature using the validator’s **public Dilithium key**.
2. **Hash Consistency Check**:
    - Nodes recompute the SHA-3 hash of the block content to ensure integrity.

```plaintext
[Validator Node]                            [Verifier Node]  
      |                                            |  
      |— 1. Propose Block with SHA-3 Hash  ———>|  
      |— 2. Dilithium-Signed Header ———————>|  
      |                                            |— 3. Verify Signature & Hash  
      |                                            |— 4. Append to Blockchain  
```

## Technical Details

| Parameter          | Dilithium2 | Dilithium3 | Dilithium5 |
|-------------------|------------|------------|------------|
| **Security Level** | 128-bit    | 192-bit    | 256-bit    |
| **Public Key Size** | 1,312 bytes | 1,952 bytes | 2,592 bytes |
| **Private Key Size** | 2,528 bytes | 4,000 bytes | 4,864 bytes |
| **Signature Size** | 2,420 bytes | 3,293 bytes | 4,595 bytes |
| **Signing Speed (ops/sec)** | ~1,000 | ~700 | ~500 |
| **Verification Speed (ops/sec)** | ~10,000 | ~8,000 | ~6,000 |

## Integration with Substrate

#### Pallet Design
```rust
#[pallet::config]
pub trait Config: frame_system::Config {
    type Signature: Member + Verify<Self::PublicKey> + From<DilithiumSignature>;
    type PublicKey: Member + IdentifyAccount<AccountId = Self::AccountId>;
}
```

## Comparison with Classical Algorithms

| Algorithm  | Key Size (Bytes) | Signature Size (Bytes) | Quantum Resistance | Signing Speed (ops/sec) |
|------------|-----------------|-----------------|------------------|------------------|
| **ECDSA**  | 32              | 64              | ❌               | ~50,000          |
| **SR25519** | 32             | 64              | ❌               | ~45,000          |
| **Dilithium2** | 1,312        | 2,420          | ✅               | ~1,000           |

## Security Considerations

- **NTRU Lattice Hardness (FALCON)**: Best-known attacks require solving the Shortest Vector Problem (SVP) on NTRU lattices, which is infeasible for quantum computers.
- **Signature Malleability (FALCON)**: Mitigated by hashing transactions with SHAKE-256 before signing.
- **Key Management**: Private keys are encrypted using AES-256-GCM in the wallet layer.

### Resources
- **[Dilithium Paper](https://pq-crystals.org/dilithium/data/dilithium-specification-round3-20210201.pdf)**
- **[Code repository](https://github.com/pq-crystals/dilithium)**
- **[Komodo Integrates Dilithium](https://komodoplatform.com/en/blog/dilithium-quantum-secure-blockchain/)**
- **[NIST PQC Project](https://csrc.nist.gov/projects/post-quantum-cryptography)**
- **[Substrate Developer Hub](https://substrate.dev/docs/en/)**