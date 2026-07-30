# What is stored

Short version: an address, a salted fingerprint, a timestamp, and a proof
reference. Nothing else.

---

## What we keep

```json
{
  "fingerprint": "e55e97d2890aeac52cea13950559b97a95f121c31b0fe809ba3b3029db1204f2",
  "address": "0x9ccD05763D3b3C49eA1daF33392eA9C3E5fA9c4A",
  "source": "binance",
  "taskId": "0x1d44051166388ce2f92ac83a95d4313614c5ed65f7979b2c1ba6dbc8c0a78f65",
  "at": "2026-07-30T16:45:20.349Z"
}
```

That is a real row. There is no other table.

The `fingerprint` is `HMAC-SHA256(secret_salt, account_id)`. It exists to notice
that the same exchange account came back with a different wallet, nothing more.
It cannot be reversed, and without the salt it cannot be brute-forced either,
which matters because account ids are short and guessable.

## What we never receive

- exchange passwords, cookies, session tokens
- names, dates of birth, addresses, nationality
- identity documents or selfies
- balances, trades, positions, transaction history

The zkTLS proof only ever covers the fields the template names. For the KYC
templates that is the verification level and the account id. There is no
mechanism by which anything else could reach us.

## What you receive as a platform

A boolean, which exchange, when, and a `taskId`.

You cannot obtain the account id, the fingerprint, or anything about the person
through the API. It is not a permission setting. The data is not there.

## What is public on Base

The attestation is reported on-chain, permanently, by design. That is what makes
it independently checkable.

It contains the exchange endpoint that was queried, the KYC level, the exchange
account id, the wallet address, and the attestor's signature.

**So the link between an exchange account id and a wallet address is public.**

The id is a pseudonymous number (`48372910`, not a name) and reading it tells
you nothing about who that person is unless you already work at the exchange. But
it is permanent and it cannot be deleted afterwards, and users deserve to know
that before they start rather than after.

We chose this deliberately: it is the same property that lets you verify our
answers without trusting us.

## Deletion

Ask and we remove your row from the allowlist, which frees the slot.

We cannot remove the on-chain attestation. Nobody can. That is what a blockchain
is. This is worth saying plainly rather than promising a deletion we cannot
perform.

## Salt rotation

Rotating the salt invalidates every existing fingerprint, so a previously
verified person could take a second slot. We therefore do not rotate it, and it
is stored separately from the allowlist.

If it ever leaked, the exposure is that someone holding both the salt and a guess
at an account id could confirm the guess. They still could not enumerate ids from
the file.
