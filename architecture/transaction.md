# Quantum-Resistant Blockchain: Transaction Layer

## **Purpose**
The transaction layer ensures the **integrity and authenticity** of user-initiated operations. It replaces classical 
signature schemes (e.g., ECDSA, RSA) with **FALCON**, a lattice-based algorithm, to secure transactions against quantum 
attacks while maintaining high throughput.

---

## **Why FALCON?**
### **Algorithm Overview**
- **Type**: NTRU lattice-based digital signature scheme.
- **Security Foundation**: Based on the **NTRU lattice problem**, which is resistant to both classical and quantum attacks.
- **NIST Standardization**: Part of the NIST Post-Quantum Cryptography Project’s **Round 3 finalists** (Category: Digital Signatures).

### **Quantum Resistance**
- **Shor’s Algorithm Immunity**: NTRU lattices are not vulnerable to quantum attacks targeting integer factorization or discrete logarithms.
- **Compact Signatures**: Achieves small signature sizes (e.g., 666 bytes) compared to other post-quantum schemes (e.g., SPHINCS+ signatures are ~41 KB).

---

## **Workflow: Transaction Signing and Validation**
### **Step 1: Transaction Creation**
1. **User Input**: A transaction is created (e.g., token transfer, smart contract call).
2. **Message Digest**: The transaction data is hashed using **SHAKE-256** (extendable-output function from the SHA-3 family).

### **Step 2: Transaction Signing**
1. **Private Key Usage**:
    - The user signs the transaction digest using their **private FALCON key**.
2. **Signature Generation**:
    - FALCON leverages **Fast Fourier Transform (FFT)** optimizations to generate compact lattice-based signatures.

### **Step 3: Transaction Validation**
1. **Public Key Verification**:
    - Nodes verify the signature using the user’s **public FALCON key**.
2. **Hash Recalculation**:
    - Nodes recompute the SHAKE-256 hash of the transaction data to ensure integrity.

```plaintext
[User Wallet]                              [Node]  
      |                                          |  
      |— 1. Transaction Data + SHAKE-256 Hash →|  
      |— 2. FALCON-Signed Transaction ——————→|  
      |                                          |— 3. Verify Signature & Hash  
      |                                          |— 4. Add to Mempool  

```
# FALCON Integration in Substrate

## Technical Details

### FALCON

| Parameter          | FALCON-512 | FALCON-1024 |
|-------------------|------------|------------|
| **Security Level** | 128-bit    | 256-bit    |
| **Public Key Size** | 897 bytes  | 1,793 bytes |
| **Private Key Size** | 1,281 bytes | 2,561 bytes |
| **Signature Size** | 666 bytes  | 1,280 bytes |
| **Signing Speed (ops/sec)** | ~10,000 | ~5,000 |
| **Verification Speed (ops/sec)** | ~40,000 | ~20,000 |

## Integration with Substrate

### Runtime Module Design

#### FALCON Transaction Validity
```rust
impl fp_self_contained::SelfContainedCall for Call {
    fn validate_self_contained(&self) -> Option<Result<(), TransactionValidityError>> {
        if let Call::submit_transaction { signature, public_key, .. } = self {
            // Verify FALCON signature using public key
            let is_valid = falcon_verify(&signature, &public_key, &payload);
            Some(if is_valid { Ok(()) } else { Err(InvalidTransaction::BadProof.into()) })
        } else {
            None
        }
    }
}
```

### Key Storage
- **FALCON**: Keys are stored in Substrate’s keystore using `sp_core::crypto::CryptoTypePublicPair`.
- **Dilithium**: Validators generate Dilithium keys during onboarding using Substrate’s `sp_application_crypto` crate.

## Performance Optimization

### FALCON
- **Batch Verification**: Transactions are grouped and verified in parallel to offset FALCON’s verification overhead.
- **Hardware Acceleration**: Leverage AVX2/NEON instructions for FFT operations in FALCON.

## Comparison with Classical Algorithms

| Algorithm  | Key Size (Bytes) | Signature Size (Bytes) | Quantum Resistance | Signing Speed (ops/sec) |
|------------|-----------------|-----------------|------------------|------------------|
| **ECDSA**  | 32              | 64              | ❌               | ~50,000          |
| **RSA-2048** | 256            | 256            | ❌               | ~1,000           |
| **FALCON-512** | 897          | 666            | ✅               | ~10,000          |

## Security Considerations

- **NTRU Lattice Hardness (FALCON)**: Best-known attacks require solving the Shortest Vector Problem (SVP) on NTRU lattices, which is infeasible for quantum computers.
- **Signature Malleability (FALCON)**: Mitigated by hashing transactions with SHAKE-256 before signing.
- **Key Management**: Private keys are encrypted using AES-256-GCM in the wallet layer.
