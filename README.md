# cryptochaos-review

> A simple review and revised architecture of [CryptoChaos (arXiv:2504.08618)](https://arxiv.org/abs/2504.08618),
> a hybrid chaos-based cryptographic framework.

---

## TL;DR

The original CryptoChaos paper is genuinely interesting.
It is also probably wrong about the most interesting part.

The chaos layer - four chaotic maps mixed into the key schedule -
**likely contributes nothing to security**, and the paper's
"post-quantum" claim is undermined by the use of X25519,
which Shor's algorithm breaks.

This review:
- Identifies the concrete problems in the original design
- Proposes a revised architecture (ML-KEM + HKDF + AEAD)
- Treats the chaos layer honestly: as a research subject,
  not a security feature
- Is brutally honest about what we don't know

## Read the Paper

**[PAPER.md](./PAPER.md)** - full review and revised architecture
