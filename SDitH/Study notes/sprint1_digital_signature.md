# Sprint 1: Digital Signature

> In this sprint, I have managed to grasp basic knowledge about public/secret keys and the combination of three types of algorithm in the encrypting process.

**Mechanism:**

> When a message is on the way to its destination, it can be intercepted and its content might be exposed and tamperable. We can solve the problem by sending the message via a fully private channel, or implement security methods so that even when attackers intercept our messages, they cannot exploit. 

> A fully private channel is expensive and we cannot afford it to everyone. So people came up with an idea of using a pair of keys. This pair of keys include a secret key (used by senders, to sign a message) and a public key (used be receivers, to verify the message).

> To use this pair of keys, we need need 3 algorithms (signature scheme):

| Algorithm | Input | Output | Who runs this? |
| --- | --- | --- | --- |
| Keygen | security paramter $\lambda$; a random element | (pk, sk) | Sender (once) |
| Sign | sk, message m | signature $\sigma$ | Sender (when sending a message) |
| Verify | pk, m, $\sigma\$ | true/false | Receivers | 

**An example (RSA with very small n):**

> Keygen:   
p = 61, q = 53, n = pq = 3233, e = 17  
-> find d so that $e.d \equiv 1 \pmod{(p-1)(q-1)}$  
-> d = 2753  
-> **pk = (n, e) = (3233, 17); sk = d = 2753**

> Sign:  
h = hash(m) mod n = 724
-> $\sigma$ = $h^d$ mod n = 724^2753 mod 3233   
-> **$\sigma$ = 2464**

> Verify:  
h' = hash(m_receive) mod n = 724  
$\sigma^e$ mod n = 724  
-> MATCH

> If an attacker can derive p, q from n, he/she can easily calculate d from (n, e). 
So the strength of the encryption equals to how hard it is to factorize n. 
In reality, RSA use n >= 2048, so it **seems** to take forever to get p, q from n.

> Different schemes apply a different "hard problem":

| Scheme | Hard problem | 
| --- | --- |
| RSA | Integer factorization |
| ECDSA, Ed25519 | Elliptic curve discrete logarithm | 
| Dilithium | Lattice problem | 
| SDitH | Syndrome decoding | 

> To compare these schemes, we use 3 parameters:
> 1. **Size** of pk, sk and signature (bytes)
> 2. **Speed** of the 3 algorithms
> 3. **Resources** when making hardware

| Scheme (~128-bit level) | pk (bytes) | Signature (bytes) |
| :--- | :---: | :---: |
| RSA-3072 | 384 | 384 |
| ML-DSA-44 (Dilithium) | 1 312 | 2 420 |
| SPHINCS+-128s | 32 | 7 856 |
| SDitH L1 | 120 | <= 8 260 |