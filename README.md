<p align="center">
  <a href="http://www.adjoint.io"><img src="https://www.adjoint.io/images/logo-small.png" width="250"/></a>
</p>

[![CircleCI](https://circleci.com/gh/adjoint-io/aos-signature.svg?style=svg&circle-token=ec783d4839d6a26e274796dd6e014399eac3918b)](https://circleci.com/gh/adjoint-io/aos-signature)

A ring signature, also know as a Spontaneous Anonymous Group (SAG) or 1-out-of-n signature, convinces a verifier that a message is signed by any member in a group of n independent signers without allowing the verifier to identify who the signer was.

Abe-Ohkubo-Suzuki Ring Signatures
=================================

In their paper, "1-out-of-n Signatures from a Variety of Keys"[1], Abe, Ohkubo and Suzuki (AOS) present a method to construct a 1-out-of-n signature scheme that allows mixture use of different flavours of keys at the same time.

Linkable Spontaneous Anonymous Group (LSAG) Signature
=====================================================

Liu, et al.[2] add the property of linkability to ring signatures. Linkability means
that two signatures by the same signer can be identified as such, but the signer remains anonymous. It adds the feature of claimability, which allows a signer to claim responsibility by providing proof of having generated a given signature.

A LSAG signature scheme satisfies three properties:

- **Anonymity**: A signer cannot be distinguished from a pool of `t` commitments (public keys).
- **Spontaneity**: No group secret, group manager of secret sharing setup stage.
- **Linkability**: Two signatures by the same signer can be linked.

A LSAG Signature Scheme over elliptic curves
============================================

It consists of two parts: signature generation and signature verification. Let L = {y<sub>0</sub>, ..., y<sub>t-1</sub>} be a list of `t` public keys. Let H:{0, 1}* -> Z<sub>n</sub> where `H` is a cryptographic hash function and `n` is the order of the elliptic curve over a finite field F<sub>q</sub>. For i ∈ {0, ..., t-1},
each user `i` has a distinct public key y<sub>i</sub> and a private key x<sub>i</sub>.

### Signature Generation

Let k ∈ {0, ..., t-1} be the position of the prover's public key in the list `L`
of public keys. Let x<sub>k</sub> be its private key. The LSAG signature of a message m ∈ {0,1}* is generated by the following steps:

1. Compute h = [H(L)] \* g, where `g` is the generator of the elliptic curve, and
y = [x<sub>k</sub>] \* h. Both computations are the product of a scalar and a point in the curve.

2. Select u ∈ Z<sub>n</sub> and compute the first challenge ch<sub>k+1</sub> = H(L, y, m, [u] \* g, [u] \* h)

3. For i in {k+1, ..., t-1, 0, ... k-1}, choose s<sub>i</sub> ∈ Z<sub>n</sub> and compute the remaining challenges: ch<sub>i+1</sub> = H(L, y, m, [s<sub>i</sub>] \* g + [ch<sub>i</sub>] \* y<sub>i</sub>, [s<sub>i</sub>] \* h + [ch<sub>i</sub>] \* y)

4. With the last ch<sub>k</sub> computed, calculate s<sub>k</sub> = (u - x<sub>k</sub> \* ch<sub>k</sub>) mod n

The signature is (ch<sub>0</sub>, [s<sub>0</sub>, ..., s<sub>t-1</sub>], y).

### Signature Verification

Given a message `m`, a signature of a message (ch<sub>0</sub>, [s<sub>0</sub>, ..., s<sub>t-1</sub>], y) and a list of public keys `L`, an honest verifier checks a signature as follows:

1. For i in {0, ..., t-1} compute ch<sub>i+1</sub> = H(L, y, m, [s<sub>i</sub>] \* g + [ch<sub>i</sub>] \* y<sub>i</sub>, [s<sub>i</sub>] \* h + [ch<sub>i</sub>] \* y), where h = [H(L)] \* g.

2. Check whether c<sub>0</sub> is equal to H(L, y, m, [s<sub>t-1</sub>] \* g + [ch<sub>t-1</sub>] \* y<sub>t-1</sub>, [s<sub>t-1</sub>] \* h + [ch<sub>t-1</sub>] \* y)

```haskell
testSignature
  :: ECC.Curve
  -> Int
  -> ByteString
  -> IO Bool
testSignature curve nParticipants msg = do
  -- Generate public and private keys
  (pubKey, privKey) <- ECC.generate curve
  -- Generate random foreign participants
  extPubKeys <- genNPubKeys curve nParticipants
  -- Position of the signer's key in the set of public keys
  k <- fromInteger <$> generateBetween 0 (toInteger $ length extPubKeys - 1)
  -- List of public keys
  let pubKeys = insert k pubKey extPubKeys
  -- Sign message with list of public keys and signer's key pair
  signature <- sign pubKeys (pubKey, privKey) k msg
  -- Verify signature
  pure $ verify pubKeys signature msg
```

**References**:
1. M. Abe, M. Ohkubo, K. Suzuki. "1-out-of-n Signatures from a Variety of Keys", 2002
2. K. Liu, K. Wei, S. Wong. "Linkable Spontaneous Anonymous Group
Signature for Ad Hoc Groups", 2004


**Notation**:

1. `[b] * P`: multiplication of a point P and a scalar b over an elliptic curve defined over a finite field modulo a prime number

License
-------

```
Copyright Adjoint Inc. (c) 2018

All rights reserved.
```
