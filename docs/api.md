# API reference

Base URL: `https://api.zenid.app`

All endpoints require a bearer token.

```
Authorization: Bearer zen_live_...
```

Requests without a valid key get `401`. Keys are hashed at rest. If you lose
yours we issue a new one, we cannot read the old one back.

---

## `GET /v1/status`

Whether an address has earned a slot.

| Parameter | Required | |
| --- | --- | --- |
| `address` | yes | EVM address, any case |

```bash
curl 'https://api.zenid.app/v1/status?address=0x9ccD0576...' \
  -H 'Authorization: Bearer zen_live_...'
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

| Field | |
| --- | --- |
| `verified` | The only field you need to gate on |
| `source` | `binance` or `okx`, `null` if unverified |
| `verifiedAt` | ISO 8601, `null` if unverified |
| `taskId` | Attestation on the Primus task contract. Check it yourself |
| `chainId` | `8453` Base, `84532` Base Sepolia |

An unverified address is not an error:

```json
{ "address": "0x2222...", "verified": false, "source": null, "verifiedAt": null, "taskId": null, "chainId": 8453 }
```

---

## `GET /v1/allowlist`

Every verified address. Use it to sync your own database instead of polling one
address at a time.

| Parameter | Required | |
| --- | --- | --- |
| `since` | no | ISO 8601. Only rows verified after this |

```bash
curl 'https://api.zenid.app/v1/allowlist?since=2026-07-01T00:00:00Z' \
  -H 'Authorization: Bearer zen_live_...'
```

```json
{
  "count": 2,
  "entries": [
    { "address": "0x9ccD0576...", "source": "binance", "verifiedAt": "2026-07-30T16:45:20.349Z", "taskId": "0x1d4405..." },
    { "address": "0x7F58Bd09...", "source": "okx",     "verifiedAt": "2026-07-30T18:02:11.882Z", "taskId": "0xa9948e..." }
  ]
}
```

Store `verifiedAt` from your last sync and pass it back as `since` on the next
one.

---

## Restricting a campaign to one exchange

A person with both a Binance and an OKX account can take two slots with two
wallets. Nothing links those accounts, so this cannot be detected.

If that matters for your campaign, accept a single source:

```js
const res = await fetch(`https://api.zenid.app/v1/status?address=${address}`, {
  headers: { Authorization: `Bearer ${process.env.ZENID_KEY}` },
}).then((r) => r.json());

const eligible = res.verified && res.source === 'binance';
```

You lose the users who only have OKX. Your call.

---

## Errors

| Status | |
| --- | --- |
| `400` | Malformed request. `{ "error": "invalid address" }` |
| `401` | Missing or unknown API key |
| `429` | Rate limited |
| `5xx` | Our problem. Retry with backoff |

Errors always carry a JSON body:

```json
{ "error": "invalid address" }
```

---

## Rate limits

600 requests per minute per key. Ask if you need more. The limit exists to catch
runaway loops, not to meter you.

`GET /v1/allowlist` with `since` costs one request whatever the row count, so
prefer it over looping `status`.

---

## Examples

**Node**

```js
async function isVerified(address) {
  const r = await fetch(`https://api.zenid.app/v1/status?address=${address}`, {
    headers: { Authorization: `Bearer ${process.env.ZENID_KEY}` },
  });
  if (!r.ok) throw new Error(`zenid ${r.status}`);
  return (await r.json()).verified;
}
```

**Python**

```python
import os, requests

def is_verified(address: str) -> bool:
    r = requests.get(
        "https://api.zenid.app/v1/status",
        params={"address": address},
        headers={"Authorization": f"Bearer {os.environ['ZENID_KEY']}"},
        timeout=10,
    )
    r.raise_for_status()
    return r.json()["verified"]
```

**Bulk sync**

```js
async function sync(since) {
  const url = `https://api.zenid.app/v1/allowlist${since ? `?since=${since}` : ''}`;
  const { entries } = await fetch(url, {
    headers: { Authorization: `Bearer ${process.env.ZENID_KEY}` },
  }).then((r) => r.json());

  for (const e of entries) await db.upsert(e.address, e);
  return entries.at(-1)?.verifiedAt ?? since;
}
```
