# What happens during a verification

Roughly thirty seconds, five steps. The user signs one message and pays nothing.

---

### 1 — The user picks an exchange

Binance or OKX. They need an account that passed **KYC level 2**, meaning
document verification, not just a confirmed email.

Level 1 is a name and a date of birth and can be opened in bulk, so it is
rejected. This threshold is why a slot is worth something.

### 2 — The user proves the wallet

They sign a short message with the wallet they want allowlisted.

This is a signature, not a transaction: no gas, no funds, no network switch, and
it works on any chain their wallet happens to be on.

Without it anyone could allowlist a wallet they do not control and burn its
eligibility.

### 3 — ZENID pays for the attestation

We submit and fund the verification task on Base ourselves.

This is the reason your users need no ETH and no wallet balance. Most of them
would drop out at that step, so we absorb it.

### 4 — zkTLS reads the KYC status

The Primus browser extension opens a session with the exchange and produces a
cryptographic proof of the response, taken from the encrypted TLS transcript.

An attestor — selected at random, running inside a Phala TEE — signs the result
and reports it to the Primus task contract on Base.

Three things worth being precise about:

- The exchange sees an ordinary logged-in session. It is never told why.
- Credentials and cookies never leave the user's browser. Not to us, not to the attestor.
- Neither we nor Primus can forge the signature.

### 5 — The slot is recorded

Our server ignores whatever the browser claims and reads the attestation back
from the chain, then checks that:

- the task was one we paid for
- it used the expected template
- it was issued to the address being claimed
- an attestor actually reported a result
- the KYC level clears the threshold

Only then does the account id get hashed with `HMAC-SHA256` under a secret salt.
We keep the fingerprint and the address. The id itself is never written down.

A second wallet presenting the same account produces the same fingerprint and is
refused.

---

## Why the browser extension

The user's browser terminates TLS internally and hands JavaScript only the
decrypted body. A web page therefore cannot see the encrypted transcript that the
proof is built from, and same-origin policy stops it reading another domain's
authenticated session at all.

An extension sits outside that sandbox, which is what makes zkTLS possible in a
browser. This is a constraint of the web platform, not a design choice — every
zkTLS provider has it.

The practical consequence: **verification is desktop only.** Chrome and Safari on
mobile do not support extensions.

## What we do when it fails

Nothing is recorded and no slot is consumed. The user can retry.

A failure part way through costs us the attestation fee, not the user, and not
you.
