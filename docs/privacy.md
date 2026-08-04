# What is stored

Short version: an address, a salted fingerprint, a timestamp, and a proof
reference. Nothing else.

---

## What we keep

```json
{
  "fingerprint": "e55e97d2890aeac52cea13950559b97a95f121c31b0fe809ba3b3029db1204f2",
  "address": "0x9ccD05763D3b3C49eA1daF33392eA9C3E5fA9c4A",
  "source": "okx",
  "level": 2,
  "proof": "0x09ec954208a3291ca8aa5d18b984efb352fce1c648f33610fc5824cb39da12d0...",
  "at": "2026-07-30T16:45:20.349Z"
}
```

That is a real row. There is no other table.

The `fingerprint` is `HMAC-SHA256(secret_salt, account_id)`. It exists to notice
that the same exchange account came back with a different wallet, nothing more.
It cannot be reversed, and without the salt it cannot be brute-forced either,
which matters because account ids are short and guessable.

The `level` is the verification tier the exchange reported, kept so a platform
can ask for more than our own threshold. It is a small integer on OKX and `null`
on Binance, which publishes no numeric scale. It says nothing about the person
beyond how far they went through their exchange's own checks.

## What we never receive

- exchange passwords, cookies, session tokens
- names, dates of birth, addresses, nationality
- identity documents or selfies
- balances, trades, positions, transaction history

The zkTLS proof only ever covers the fields the template names. For the KYC
templates that is the verification level and the account id. There is no
mechanism by which anything else could reach us.

## What you receive as a platform

A boolean, which exchange, when, and the attestor signature.

You cannot obtain the account id, the fingerprint, or anything about the person
through the API. It is not a permission setting. The data is not there.

## Nothing is published on chain

The attestation is signed by the attestor and handed to us. It is never written
to a public ledger.

So the exchange account id never becomes public, and the link between an exchange
account and a wallet address stays between the attestor and us. We reduce it to a
salted fingerprint the moment we receive it.

This is a change from an earlier design where every attestation was reported
on-chain. It made verification easy but published the account id forever, which
was too high a price.

Verifiability did not suffer. The attestor signature is still checkable by anyone
we hand the attestation to, with the Primus SDK or their verifier contract.

## The attestation itself

We never store it. It reaches our server, gets checked, and is dropped. Keeping
it would mean keeping the account id, which is the one thing this design refuses
to do.

The user can download it at the end of their verification, straight from the
browser. They hold the only copy, and they decide who ever sees it. They can
also download a receipt with no account id in it, safe to hand around.

One consequence worth stating: if a user loses their attestation, nobody can
reissue it. They would have to verify again.

## Deletion

Ask and we remove your row from the allowlist, which frees the slot. There is
nothing else to delete, and nothing outside our control that would survive it.

## Salt rotation

Rotating the salt invalidates every existing fingerprint, so a previously
verified person could take a second slot. We therefore do not rotate it, and it
is stored separately from the allowlist.

If it ever leaked, the exposure is that someone holding both the salt and a guess
at an account id could confirm the guess. They still could not enumerate ids from
the file.
