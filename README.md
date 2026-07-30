# ZENID

**One slot per person.** ZENID lets your users prove they hold a KYC-verified
account on a major exchange, and gives you back a yes or a no.

Use it anywhere one person should count once: airdrops, quest campaigns, mints,
governance, grants, community roles.

You never receive a name, a document, or an account number. You receive a boolean
and a proof reference you can check yourself.

- **Site**: [zenid.app](https://zenid.app)
- **Attestations**: [Primus zkTLS](https://primuslabs.xyz), settled on Base
- **This repo**: integration docs only. The service itself is closed source.

---

## The problem

In crypto an identity costs nothing. A wallet is a keypair, and one person can
hold ten thousand of them.

So every mechanism that hands something out per address gets farmed. An airdrop
with 40,000 claimants usually has 4,000 humans behind it. A quest campaign pays
for engagement and buys scripts. A quadratic matching pool meant to reflect a
community gets redirected by a handful of fake ones. A mint sells its supply to a
bot army in the first block.

It is the same failure every time: **you needed one person to count once, and you
had no way to tell.**

The obvious fix is KYC, and nobody wants it. Your users refuse to send passports
to a random protocol, and you do not want to hold identity documents you are then
legally responsible for.

## What ZENID does instead

Your users already passed KYC, at Binance, at OKX. ZENID proves that fact
without moving the underlying data.

The user's browser opens a verified session with their exchange. A cryptographic
proof of the response is produced and signed by an independent attestor. We read
that proof, check the KYC level, and record a **salted fingerprint** of the
account id.

We never see credentials. We never store the account id. You never touch either.

## What you get

| | |
| --- | --- |
| One person | One slot, whatever wallet they try next |
| Your data liability | Zero, no identity ever reaches you |
| User cost | Zero, no gas, no transaction, no fee |
| Verification time | About 30 seconds |

---

## Where this is used

Anywhere one person can create a thousand identities and profit from it.

**Airdrops.** The obvious one. Farmers run hundreds of wallets and take the
share meant for your community. Gate the claim on a verified slot and their
budget becomes one KYC-verified exchange account per wallet.

**Quest and points campaigns.** Galxe, Zealy, Layer3 style programs are farmed
at industrial scale. The points you pay for engagement mostly buy scripts.

**NFT mints and allowlists.** A bot army takes the supply in the first block and
your actual community pays secondary prices.

**Testnet incentive programs.** You pay for usage signal and receive noise, then
have to guess afterwards who was real.

**Retroactive funding and grants.** Quadratic matching only works if one person
means one voice. A handful of sybils redirects the whole matching pool.

**Governance.** One person one vote instead of one token one vote, for anything
where wealth should not decide alone.

**Faucets and rate limits.** Any free resource where creating a new identity
costs nothing to abuse.

**Discord and community roles.** A Verified Human role that a bot cannot obtain.

**Undercollateralized lending.** Lending below full collateral requires knowing
the borrower is one person and not a fresh wallet. This is the identity half of
that problem.

## Where this does not help

Worth saying plainly so you do not waste your time.

It proves **personhood, not honesty**. A verified person can still behave badly,
they just cannot do it a thousand times in parallel.

It does not prove **wealth, skill or history**. Only that a KYC-verified account
stands behind the address.

It does not stop someone verifying a wallet for a friend. No system does. What it
does is make every fake slot cost a real KYC-verified exchange account.

---

## Integrating

Three steps.

**1. Send your user to ZENID**

```
https://zenid.app/?ref=YOUR_PLATFORM_ID&return=https://yourapp.com/verified
```

They pick their exchange, connect their wallet, and verify. We bring them back to
your `return` URL when it is done.

**2. Ask us about the address**

```bash
curl https://api.zenid.app/v1/status?address=0x9ccD05763D3b3C49eA1daF33392eA9C3E5fA9c4A \
  -H "Authorization: Bearer zen_live_..."
```

```json
{
  "address": "0x9ccD05763D3b3C49eA1daF33392eA9C3E5fA9c4A",
  "verified": true,
  "source": "binance",
  "verifiedAt": "2026-07-30T16:45:20.349Z",
  "taskId": "0x1d44051166388ce2f92ac83a95d4313614c5ed65f7979b2c1ba6dbc8c0a78f65",
  "chainId": 8453
}
```

**3. Gate on `verified`**

That is the whole integration. Full reference in [docs/api.md](docs/api.md).

---

## Don't trust us

Every response carries a `taskId`. That is a real task on the Primus task
contract on Base, holding the attestation, the attestor that signed it, and its
timestamp.

Read it yourself:

```js
const task = new ethers.Contract(
  '0x151cb5eD5D10A42B607bB172B27BDF6F884b9707',
  ['function queryTask(bytes32) view returns (tuple(string templateId, address submitter, address[] attestors, tuple(address attestor, bytes32 taskId, tuple(address recipient, tuple(string url, string header, string method, string body)[] request, tuple(tuple(string keyName, string parseType, string parsePath)[] oneUrlResponseResolve)[] responseResolve, string data, string attConditions, uint64 timestamp, string additionParams) attestation)[] taskResults, uint64 submittedAt, uint8 tokenSymbol, address callback, uint8 taskStatus))'],
  provider,
);

const info = await task.queryTask(taskId);
```

You will find the exchange endpoint that was queried, the attestor's address, and
the KYC level returned. If our answer and the chain disagree, the chain is right.

No other verification provider lets you do this.

---

## Getting an API key

Keys are issued manually while ZENID is in early access.

Reach out on any of these:

| | |
| --- | --- |
| Issue | Open one on this repo |
| X | [@krimdevnode](https://x.com/krimdevnode) |
| Discord | `niceone1809` |
| Email | `contact@zenid.app` |

Tell us:

- your platform or protocol name
- what you are gating (airdrop, allowlist, governance, something else)
- rough number of users you expect to verify

You get a `zen_live_...` key back. It is shown once and stored only as a hash, so
save it when you receive it, we cannot recover it, only replace it.

---

## Docs

- [docs/api.md](docs/api.md): endpoints, errors, rate limits
- [docs/flow.md](docs/flow.md): what happens during a verification, step by step
- [docs/privacy.md](docs/privacy.md): exactly what is stored and what is not
