# @ — Dogecoin Handles

`@jon` is a Doginals inscription. The first valid inscription wins, and holding that inscription output is owning the handle. The carrier is an `ord` inscription (P2SH commit → reveal), the same carrier as a `.doge` name. It is not an OP_RETURN, and it is not a `.doge` name with a new suffix.

| | |
| ---: | --- |
| **Version** | v1 |
| **Class** | Transferable handle object |
| **Payload** | Doginals inscription: plain `@jon` or JSON `"p": "at"` |
| **User-facing name** | @ handles · `@jon` |
| **Protocol / wire identifier** | `p: "at"` · plain text `@` + localpart |
| **Source of truth** | [spec.md](spec.md) |
| **Indexer** | dogex `/api/at/*` (specified; ship against this document) |
| **Product** | [doginals.lol/names](https://doginals.lol/names) — handles beside `.doge` names |
| **Wallet** | Dig — recipient `@jon` resolves this inscription, not a Ð𝕏 bind |
| **Public copy** | <https://github.com/jonheaven/at> |

### Why this is not DNS and not Ð𝕏

| | **@ (this)** | **DNS** | **Ð𝕏** |
| --- | --- | --- | --- |
| What you hold | `@jon` | `jon.doge` | nothing — a statement |
| Body | exact `@jon` | `label.namespace` (one dot, no `@`) | OP_RETURN `DX` or `p:"dx"` |
| Proof | the inscription | the inscription | a public X post that contains the address |
| Transfer | spend the inscription | spend the inscription | no — revoke frees the claim |
| Owns the X account? | no | no | claims it, until revoke |

Minting `jon.at` would be a DNS name in an open namespace. It would not be `@jon`. Putting `@jon` inside DNS would require allowing `@`, which DNS rejects on purpose. Putting it inside Ð𝕏 would make the handle revocable and dependent on an X post, which is not ownership.

Bitcoin’s closest clean ancestors are [Sats Names](https://docs.satsnames.org/) (first inscription of the name wins, the ordinal is the name) and plain-text `.sat` names. Sidechain placeholders (a Rootstock handle with a Bitcoin “proof” inscription) are a registry with extra steps. This spec does not do that.

No premine. This document does not assign `@jon` or any other handle to any address.
