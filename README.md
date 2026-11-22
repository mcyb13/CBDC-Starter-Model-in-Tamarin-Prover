# CBDC Starter Model in Tamarin

A small symbolic model of a central bank digital currency (CBDC) payment, built in the [Tamarin prover](https://tamarin-prover.com/) (tested with Tamarin 1.8.0) as a first step toward modelling privacy-preserving CBDC schemes with zero-knowledge proofs and threshold signatures.

This is a learning project. The model is deliberately small and does not claim to capture the security of a real CBDC.

## The protocol

1. **Alice** creates a fresh transaction id and sends a signed payment request (`tid, Alice, Bob, ten, signature`).
2. The **issuer** verifies Alice's signature and accepts the payment.
3. **Bob** receives a confirmation message.

The adversary is the standard Dolev-Yao network attacker: it controls the network but cannot break the cryptography. There is no rule that leaks Alice's private key.

## Files

| File | Description |
|---|---|
| `cbdc_v1_replay.spthy` | Signed payments with no issuer memory. Replay is possible. |
| `cbdc_v2_fixed.spthy` | Adds a spent-transaction check at the issuer. |

## Lemmas

| Lemma | Meaning |
|---|---|
| `executable` | A full run exists: payment accepted, then received by Bob. Sanity check that the model is not vacuous. |
| `payment_integrity` | The issuer only accepts payments that Alice initiated earlier (no forgery). |
| `no_replay` | The same payment is never accepted twice (no double spend). |

## Run it

```
tamarin-prover --prove cbdc_v1_replay.spthy
tamarin-prover --prove cbdc_v2_fixed.spthy
```

Or open the interactive GUI in the folder with the files:

```
tamarin-prover interactive .
```

## Results

| Lemma | v1 (replay) | v2 (fixed) |
|---|---|---|
| `executable` | verified | verified |
| `payment_integrity` | verified | verified |
| `no_replay` | **falsified** | verified |

In v1, Tamarin finds a trace where one signed request, sent by the attacker twice, is accepted by the issuer twice. A valid signature shows who sent a request, but not that the request is new.

## What I learned while building it

- My first version had a `Pending(tid)` fact shared between Alice's rule and the issuer's rule. All lemmas verified, but only because of that internal fact. The signature check was never used in the proofs. I removed it so that integrity depends on signature unforgeability, and replay then became visible.
- In v2, `no_replay` is verified because the `SpentOnce` restriction assumes the issuer keeps a ledger of spent transaction ids. The result shows that such a check closes the replay hole, not that Tamarin discovered a defence.
- Tamarin can create several keys for "Alice" unless a restriction forbids it (`OnlyOnce`). Without it, counterexamples can be confusing.

## Limitations

- One sender, one recipient, one fixed amount, no balances.
- The issuer's confirmation to Bob is unsigned, so Bob's receipt is not authenticated.
- No key-compromise rule.
- No privacy properties, which are the main goal of the planned zero-knowledge work.
- The spent-id ledger is assumed through a restriction, not implemented.
- 
👩‍💻 Author
Maham Nasir
MS Cybersecurity, COMSATS University Islamabad, Pakistan
Researcher in Post-Quantum Cryptography & Privacy-Preserving Protocols

LinkedIn

"This project is part of ongoing independent research into post-quantum cryptographic protocol design, with applications to privacy-preserving digital currency systems (CBDCs) and secure identity protocols using formal verification."
