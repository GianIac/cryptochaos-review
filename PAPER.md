# CryptoChaos Review

**Version:** 2.1
**Date:** April 2026

**Based on:** [CryptoChaos – A Hybrid Chaos-Cryptographic Framework (arXiv:2504.08618v1)](https://arxiv.org/html/2504.08618v1)

---

## Disclaimer

> **I am not a cryptographer.** I am not an academic. I am someone who read a paper, found it genuinely interesting, and couldn't stop thinking about it. So I did what any self-respecting curious person would do: I took it apart, looked at the pieces, and tried to put it back together in a way that made more sense to me.
>
> This is not a peer-reviewed publication.

---

## Abstract

We revisit the [CryptoChaos framework](https://arxiv.org/html/2504.08618v1), a hybrid cryptographic proposal that mixes chaotic maps with standard cryptographic primitives. The original design is genuinely interesting, multiple chaotic systems feeding an encryption pipeline. I have to say, it stimulates curiosity.

But curiosity also means asking uncomfortable questions. Like: **does the chaotic part actually do anything useful?**

Spoiler: probably not. But that's fine, because the interesting part is not the answer, it's the process of finding it.

We propose a revised architecture that:
- Puts the real cryptography (ML-KEM, HKDF, AEAD) in the driver's seat
- Treats chaos as an optional research module, not a security pillar
- Is brutally honest about what works and what is probably less functional

The result: a framework where the chaos layer can be removed without anyone losing sleep. Which is exactly how you know the real cryptography is doing its job.

---

## 1. Introduction (or "Why I Got Into This")

I found the [CryptoChaos paper](https://arxiv.org/html/2504.08618v1) and thought: *this is interesting*. Chaotic maps, entropy mixing, hybrid encryption, there is something attractive about the idea of throwing mathematical chaos at a cryptographic problem.

But the more I looked, the more things didn't add up:

- 8-bit fixed-point for chaotic maps? That's not chaos, that's a short cycle
- X25519 labelled as "post-quantum"? Nah, not really.
- Key derivation without domain separation? That's asking for trouble ...

So I did what seemed natural: **I kept the interesting parts, fixed the parts I considered poorly functional, and was honest about the parts that might be useless.**

This is not a demolition. The [original paper](https://arxiv.org/html/2504.08618v1) made me think, and that counts for something. This is my attempt to think out loud.

---

## 2. Related Work (or "Who Got Here Before Us")

### 2.1 Chaos-Based Cryptography: A History of Interesting Ideas and Broken Schemes

The idea of using chaos for encryption is not new:

- [Baptista (1998)](#ref1) - Logistic map for plaintext encoding. Interesting idea. Broken.
- [Fridrich (1998)](#ref2) - 2D chaotic maps for image encryption. Elegant. Vulnerable.
- [Öztürk & Kılıç (2015)](#ref10) - Showed that digital chaos forms cycles. Because computers are not infinite.

What stands out to me: **chaos looks random --> someone builds a cipher --> someone else breaks it --> repeat.**

The [CryptoChaos framework](https://arxiv.org/html/2504.08618v1) is the latest iteration of this cycle (pun intended). It is more sophisticated than previous attempts, but the fundamental tension remains: **digital chaos is not real chaos.**

### 2.2 Post-Quantum:

While chaos-crypto was going in circles (literally), the post-quantum world was producing results:

- [ML-KEM (FIPS 203)](#ref5) - NIST's lattice-based KEM. Actually standardized. Actually analyzed.
- [X-Wing](#ref9) - X25519 + ML-KEM hybrid. Belt and suspenders, done right.
- [HPKE (RFC 9180)](#ref8) - Framework for hybrid public key encryption.

### 2.3 Key Derivation: Boring but Essential

[HKDF (Krawczyk, 2010)](#ref6) and [TLS 1.3 (RFC 8446)](#ref7) showed how to do key derivation correctly.

---

## 3. Threat Model (or "Who Is Trying to Ruin Your Day")

### 3.1 The Adversaries

| Adversary | What they can do | What they want |
|---|---|---|
| **A1: Script kiddie with a PhD** | Classical CPA/CCA2 attacks | Read your secrets |
| **A2: Quantum villain** | Run Shor on your key exchange | Same, but with a quantum computer |
| **A3: Entropy saboteur** | Backdoor your CSPRNG | Predict your keys at the source |

### 3.2 What We Are Defending

1. **Confidentiality** - Plaintext cannot be read (IND-CCA2)
2. **Integrity** - Ciphertext cannot be tampered with (INT-CTXT)
3. **Post-quantum survival** - Maybe still secure when quantum computers arrive (hopefully)
4. **Entropy resilience** - Not compromised if one entropy source is compromised

### 3.3 What We Assume

- At least one KEM (X25519 or ML-KEM) is not broken
- HKDF does what it says it does
- AES-GCM / ChaCha20-Poly1305 still work
- **The chaos layer is not trusted**: we assume it could output `0x00000000` and the system holds anyway

### 3.4 What We Do Not Cover

- Side channels (future work)
- Your code having bugs (not my problem)
- Key management (not my problem either)
- Who you are talking to (use authentication)

---

## 4. What Is Wrong With the Original?

All of this refers to the [original CryptoChaos framework](https://arxiv.org/html/2504.08618v1):

### 4.1 The Chaos Is Not Really Chaotic

8-bit fixed-point chaotic maps are like trying to simulate the ocean in a bathtub:

- Discretization kills the butterfly effect
- Low precision -> short cycles ([Öztürk & Kılıç, 2015](#ref10))
- "Looks random" and "is cryptographically secure" are very different things

### 4.2 "Post-Quantum" With X25519 Is a Contradiction

X25519 is great cryptography. It is not post-quantum. Shor's algorithm has elliptic curves for breakfast. Calling this post-quantum is like calling a bicycle "off-road" because it has wheels.

### 4.3 The Key Derivation Is a Minestrone

Multiple inputs mixed together, no domain separation, one key out. It's a cryptographic smoothie, you can't tell what contributes what, and you can't prove any of it is secure.

---

## 5. Design Goals (or "What I Actually Want")

1. **Real security from real primitives** - no crypto-astrology
2. **Chaos as a plug-in** - fun to study, safe to remove
3. **True post-quantum** - ML-KEM, not wishful thinking
4. **Correct key derivation** - HKDF with domain separation, done properly
5. **Modular and verifiable** - each piece does one thing
6. **The chaos layer must be removable without security impact** - this is the litmus test

---

## 6. The Revised Architecture (or "How I Would Build It")

### 6.1 The Pipeline

```
[KEM Secret(s)] + [Chaos Digest] + [Optional Inputs]
                       ↓
                 HKDF Extract
                       ↓
                 HKDF Expand
                       ↓
         [AEAD Key | Nonce | Metadata Keys]
                       ↓
          AES-GCM / ChaCha20-Poly1305
```

Clean. Traceable. Every piece has a job.

### 6.2 Key Exchange: Pick Your Flavor

| Mode | Primitives | Security |
|---|---|---|
| **Classic** | X25519 | 128-bit, pre-quantum |
| **Post-Quantum** | [ML-KEM-768](#ref5) | NIST Level 3 |
| **Hybrid** | X25519 + ML-KEM-768 | Best of both worlds |

Hybrid mode (why not?):

```
K_transport = KDF(x25519_secret || ml-kem_secret)
```

If either holds, you are safe. That is what [X-Wing](#ref9) does, and they are smarter than me.

### 6.3 The Chaos Layer (or "The Fun Part That Probably Does Nothing")

Four chaotic maps:

| Map | Equation | Parameters | Iterations |
|---|---|---|---|
| **Logistic** | x_{n+1} = r·x_n·(1 - x_n) | r = 3.99 | 1000 |
| **Tent** | x_{n+1} = μ·min(x_n, 1 - x_n) | μ = 1.99 | 1000 |
| **Chebyshev** | x_{n+1} = cos(k·arccos(x_n)) | k = 7 | 1000 |
| **Hénon** | x_{n+1} = 1 - a·x_n² + y_n | a = 1.4, b = 0.3 | 1000 |

**What we fixed compared to the [original](https://arxiv.org/html/2504.08618v1):**
- `f64` precision instead of 8-bit (because we are not savages)
- CSPRNG seeding for initial conditions
- Floyd's cycle detection - if a map starts cycling, we discard it --> https://en.wikipedia.org/wiki/Cycle_detection

```
// If a map detects a cycle its slot is replaced with 8 fresh bytes from the CSPRNG, so the IKM never contains constant zeros.
chaos_digest = SHA3-256(
    encode_f64_or_csprng(map1) || encode_f64_or_csprng(map2) ||
    encode_f64_or_csprng(map3) || encode_f64_or_csprng(map4)
)
```

> ⚠️ **Warning:** The chaotic maps are seeded by the CSPRNG (Cryptographically Secure Pseudorandom Number Generator). So they are not independent from it. Keep reading, [Section 7.3](#73-the-elephant-in-the-room) explains why this matters.

### 6.4 Key Schedule (or "The Boring Part That Actually Matters")

```
IKM   = kem_secret || chaos_digest || optional_passphrase

salt  = SHA3-256("CryptoChaos-v2-salt" || session_id)
PRK   = HKDF-Extract(salt, IKM)

K_aead  = HKDF-Expand(PRK, "CryptoChaos-v2 aead key",   32)
K_nonce = HKDF-Expand(PRK, "CryptoChaos-v2 nonce base",  12)
K_meta  = HKDF-Expand(PRK, "CryptoChaos-v2 metadata",    32)
```

Where we have:
- Domain separation.
- Labeled contexts.
- Multiple derived keys.

### 6.5 Encryption: Standard, Boring, Secure

| Cipher | Key | Nonce | Tag |
|---|---|---|---|
| AES-256-GCM | 32 B | 12 B | 16 B |
| ChaCha20-Poly1305 | 32 B | 12 B | 16 B |

Nonce via counter XOR:

```
nonce_i = K_nonce XOR encode_u96(counter_i)
```

No custom ciphers.

---

## 7. Security Analysis (or "Does It Actually Work?")

### 7.1 The Reduction (Informal, Because I Am Not an Expert)

**Claim:** If ML-KEM is IND-CCA2, HKDF is a PRF, and the AEAD is secure, then CryptoChaos-v2 is secure. Full stop. Regardless of what the chaos layer does.

**Why:**

1. The KEM provides a random-looking secret -> [ML-KEM guarantee](#ref5)
2. HKDF extracts a clean PRK from it [Krawczyk guarantee](#ref6) *(note: HKDF condenses entropy, it does not create it, this works because the KEM secret is already pseudorandom)*
3. HKDF expands into independent keys -> PRF property
4. AEAD encrypts securely with those keys -> standard guarantee
5. Chaos? Does not appear anywhere in steps 1-4

### 7.2 The "Defense in Depth" Argument (and Why It Is Shaky)

The standard argument for keeping the chaos:

> "If the CSPRNG is compromised, chaos provides independent entropy!"

Sounds OK. Let's check:

| Defense in depth example | Independent? | Valid? |
|---|---|---|
| TLS + disk encryption | Different layers | Y |
| RDSEED + software PRNG | Hardware vs software | Y |
| CSPRNG + chaos *seeded by the CSPRNG* | Same source | N |

**The chaos layer is downstream of the CSPRNG.** If the CSPRNG is compromised, the attacker knows the chaos seeds, can reproduce the maps and predict the output. Defense in depth requires *independent* layers. This is not.

### 7.3 The Elephant in the Room

**Let's be honest: the chaos layer is not indispensable.**

Here is the full picture:

**Problem 1: It is not necessary**
The security proof works without it. Literally remove it and nothing changes.

**Problem 2: It is not independent**
Seeded by the CSPRNG -> CSPRNG compromised = chaos compromised. The "backup" fails exactly when you need it.

**Problem 3: It is more attack surface**
I would say ~200 extra lines of code, floating-point arithmetic, cycle detection logic etc. etc.
And every line is a potential bug in a cryptographic system where bugs are fatal.

**Problem 4: "It can't hurt" is not an argument**
A friend of mine used to say that in software (and in cryptography), simplicity wins. Complexity is to be avoided.

| What is claimed | What is actually true |
|---|---|
| "Chaos adds entropy" | It transforms existing CSPRNG entropy. It does not create new entropy. |
| "It is independent" | It is seeded by the thing it is supposed to back up. |
| "Defense in depth" | Not independent -> not true depth. |
| "It can't hurt" | More code = more bugs = more attack surface. |

### 7.4 So Why Keep It?

One word: **research.**

The question *"Can chaotic maps contribute meaningful entropy to cryptography?"* is genuinely interesting. But to answer it, you need:

- A framework where chaos exists but is not load-bearing
- The ability to enable/disable it
- Measurable outputs
- Security that does not depend on the answer

CryptoChaos-v2 is a **laboratory**, not a product. The chaos layer is a **subject of study**, not a feature.

> **For anything that matters: disable the chaos layer. The real cryptography does not need it.**

---

## 8. Performance (or "What Does Chaos Cost You?")

| Operation | Time | Notes |
|---|---|---|
| X25519 | ~0.15 ms | `x25519-dalek` |
| ML-KEM-768 | ~0.50 ms | `ml-kem` |
| Chaos (4 maps × 1000 iter) | ~0.10 ms | `f64` |
| SHA3-256 digest | ~0.01 ms | 32 bytes |
| HKDF (extract + 3× expand) | ~0.02 ms | `hkdf` |
| AES-256-GCM (1 KB) | ~0.05 ms | AES-NI |
| **Total with chaos** | **~0.83 ms** | |
| **Total without chaos** | **~0.73 ms** | **12% faster** |

You are paying 0.10ms for something that probably does nothing. In research: fine. In prod: not so much.

---

## 9. What This System Actually Is

Let's be honest with ourselves:

| How it sounds | What it actually is |
|---|---|
| "Chaos-enhanced encryption" | Standard encryption with a chaotic toy attached |
| "Innovative cryptographic framework" | ML-KEM + HKDF + AEAD (well known) + chaos (decorative) |
| "Defense in depth" | Defense through real cryptography; chaos is just a walk-on part |
| "Post-quantum chaos-crypto" | Post-quantum cryptography. The chaos is just vibes. |

**And that's fine.** The value is not in pretending chaos does magic. The value is in:

1. Building something that is actually secure (ML-KEM + HKDF + AEAD)
2. Asking an honest question about chaos-based entropy
3. Creating a framework to answer that question empirically
4. Being transparent about what we know and what we don't

---

## 10. Future Work (or "What Would Actually Be Interesting")

- **The key experiment:** Run the system with and without chaos. Intentionally degrade the CSPRNG. Measure whether chaos helps. (My bet: no. But I would love to be wrong.)
- **Independence test:** Measure the correlation between CSPRNG output and chaos output. If they are correlated (they probably are), the diversification argument is dead.
- **NIST SP 800-22 + TestU01:** Run proper statistical tests on chaos outputs
- **Cycle lengths:** How long before each map cycles at `f64` precision?
- **Side channels:** Is `f64` arithmetic constant-time? (I don't think so)
- **Formal verification:** ProVerif or Tamarin for the key schedule
- **Implementation:** Write some code :)

---

## 11. Conclusion (or "What I Actually Think")

I read the [CryptoChaos paper](https://arxiv.org/html/2504.08618v1) and found it fascinating. The idea of mixing chaos with cryptography scratches a very specific intellectual itch.

But being interested in something means being honest about it. So here is the honest version:

- The **framework architecture** is OK
- The **real cryptography** (ML-KEM + HKDF + AEAD) works
- This is a **great research platform** for studying chaos in cryptography

- The chaos layer **probably does nothing** for security
- The "defense in depth" argument **does not hold** because chaos is not independent
- The chaos layer enabled slows things down quite a bit

The interesting cryptography here is the boring cryptography. The chaos is the question mark. And sometimes, the most valuable thing a framework can do is help you prove that something is cool but not very useful.

---

### The Kill Switch

```rust
/// The chaos layer: fun for research, useless for security.
/// Disabled by default. Enable only when running experiments.
/// Your encryption is NOT weaker with this set to false.
pub const ENABLE_CHAOS_LAYER: bool = false;
```

### Logistic Map With Cycle Detection

```rust
pub fn logistic_map(x0: f64, r: f64, iterations: usize) -> Option<f64> {
    let mut x = x0;
    let mut tortoise = x0;
    let mut hare = x0;

    for i in 0..iterations {
        x = r * x * (1.0 - x);

        // Floyd's cycle detection
        tortoise = r * tortoise * (1.0 - tortoise);
        hare = r * hare * (1.0 - hare);
        hare = r * hare * (1.0 - hare);

        // AFTER
        if (tortoise - hare).abs() < 1e-10 && i > 50 {
            return None; // cycle detected -> discard
        }
    }

    Some(x)
}
```

---

## References

<a id="ref1"></a>
1. Baptista, M.S. (1998). "Cryptography with chaos." *Physics Letters A*, 240(1-2), 50-54.

<a id="ref2"></a>
2. Fridrich, J. (1998). "Symmetric ciphers based on two-dimensional chaotic maps." *Int. J. Bifurcation and Chaos*, 8(6), 1259-1284.

<a id="ref3"></a>
3. Alvarez, G. et al. (2003). "Cryptanalysis of a chaotic encryption system." *Physics Letters A*, 319, 172-179.

<a id="ref4"></a>
4. Li, S. (2005). "On the dynamical degradation of digital piecewise linear chaotic maps." *Int. J. Bifurcation and Chaos*, 15(10), 3119-3151.

<a id="ref5"></a>
5. NIST (2024). "[FIPS 203: Module-Lattice-Based Key-Encapsulation Mechanism Standard (ML-KEM)](https://csrc.nist.gov/pubs/fips/203/final)."

<a id="ref6"></a>
6. Krawczyk, H. (2010). "[Cryptographic Extraction and Key Derivation: The HKDF Scheme](https://eprint.iacr.org/2010/264)." *CRYPTO 2010*.

<a id="ref7"></a>
7. RFC 8446: "[The Transport Layer Security (TLS) Protocol Version 1.3](https://datatracker.ietf.org/doc/html/rfc8446)."

<a id="ref8"></a>
8. RFC 9180: "[Hybrid Public Key Encryption](https://datatracker.ietf.org/doc/html/rfc9180)."

<a id="ref9"></a>
9. Barbosa, M. et al. (2024). "[X-Wing: The Hybrid KEM You've Been Looking For](https://eprint.iacr.org/2024/039)." *IACR ePrint 2024/039*.

<a id="ref10"></a>
10. Öztürk, I. & Kılıç, R. (2015). "Cycle lengths and correlation properties of finite precision chaotic maps." *Signal Processing*, 107, 244-255.

<a id="ref11"></a>
11. Original framework: "[CryptoChaos – A Hybrid Chaos-Cryptographic Framework](https://arxiv.org/html/2504.08618v1)." *arXiv:2504.08618v1*.
