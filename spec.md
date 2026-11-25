# @ — Dogecoin Handles

**Ordering:** commit-first — the DNS §5.7 key, no activation height  
**Status:** Launch draft — registration carrier is normative. dogex, doginals.lol, and Dig implement this document; they do not define it.  
**Last Updated:** 2026-11-24  
**Chain:** Dogecoin  
**Protocol Marker:** `p: "at"` (JSON) or plain `@` + localpart  
**Public copy:** <https://github.com/jonheaven/at>

---

## 1. Introduction

An @ handle is a short name you can hold, such as `@jon`. It is a Doginals inscription. The first valid inscription of that handle wins. The holder of that inscription output is the owner. Spending the output transfers the handle.

The normal mint is plain text. The inscription body **is** `@jon`. A later `op:"config"` from that holder sets a pay-to address, a site, or a bio. The config is not a second handle.

This is a sibling of DNS, not a namespace inside it, and not a mode of Ð𝕏.

- **DNS** registers `label.namespace` (`jon.doge`). The grammar requires one dot and rejects `@`. `jon.at` is a DNS name. It is a different string from `@jon`.
- **Ð𝕏** binds an X username to a Dogecoin address with an OP_RETURN `DX` claim and a public post. The bind is not an inscription you can sell. Revoke frees it. It does not prove you hold `@jon`, and holding `@jon` does not prove the X account.
- **ÐN05** is `local@domain` for Nostr. The `@` sits in the middle. A leading `@` with no dot is not ÐN05.

Bitcoin already has name inscriptions and a few `@` products. The ones worth copying are the ones where the chain is the whole registry:

| Bitcoin thing | What it actually is | Use it? |
| --- | --- | --- |
| [Sats Names](https://docs.satsnames.org/) (`.sats`) | JSON `{"p":"sns","op":"reg","name":"…" }` on an ordinal. First inscription wins. The sat is the name. No DAO, no rent, no token. | Yes — this is the ancestor of DNS and of this spec. |
| Plain-text `.sat` | Body is `name.sat`, `text/plain` only. First inscription wins. | Yes — the plain-text mint here is this idea, with `@jon` instead of a dotted suffix. |
| [@bitcoin / Spaces](https://atbitcoin.com/) | Handle tied to a UTXO plus a certificate and signature scheme (`bob@bitcoin`). | No — a second credential format. The inscription output is already the credential. |
| HNDL | Real ownership on a Rootstock contract. The Bitcoin inscription is a placeholder that dies if the sidechain handle expires. | No — a company registry with an L1 sticker. |
| BNRP | A resolver that reads names. It does not issue them. | The shape of `/api/at/*` only. dogex reads. It does not grant handles. |

Marketplaces on Bitcoin (Gamma, Magic Eden, and the rest) do not have a handle opcode. They trade the inscription with a PSBT. A Dogecoin handle marketplace is the same: list that inscription. ÐMP already does that. This spec adds no order format.

No address is assigned a handle by this document. There is no reserved list and no premine. `@jon` belongs to whoever’s valid inscription ranks first, including one that was already confirmed before this file existed.

---

## 2. Status and Scope

- **Status:** Launch draft
- **In scope:** Handle grammar, exact plain-text register, JSON register / config / primary, FCFS, transfer-by-output, indexer API, wallet resolution against Ð𝕏
- **Out of scope:** ICANN, trademark, yearly rent, proving you control an X or Nostr account, OP_RETURN handles, a new marketplace wire
- **Compatibility:** DNS, Ð𝕏, and ÐN05 keep their bytes unchanged. An indexer that implements this spec MUST NOT write those protocols’ rows from an `@` body, and MUST NOT write an @ row from their bodies.

---

## 3. Terminology

- **Handle:** canonical `@` plus a localpart. Example: `@jon`.
- **Localpart:** the characters after `@`. 1 to 15 characters. A character is `a-z`, `0-9`, `_`, one Extended_Pictographic scalar, or one flag (two regional indicators). Examples: `jon`, `jon🐕`, `🇺🇸`.
- **Register inscription:** the FCFS inscription that created the handle. Its output **is** the handle.
- **Owner:** the address that can spend that output.
- **Config:** a later inscription from the current owner. It updates records. It is not the handle.
- **Primary:** the one handle an address chooses to wear, among those it holds.
- **Missed claim:** a later valid register of a handle that was already taken. It is not the handle.

---

## 4. Design Goals

### Goals

- `@jon` is something you hold, sell, and gift, the same way as a Doginal.
- The artifact is the string. Plain text is enough. JSON is only for records.
- First valid inscription wins. Fee is the only rate limit.
- Any indexer can recompute every handle from Dogecoin blocks alone.
- The string `@jon` cannot be a DNS name or a Ð𝕏 bind by accident.

### Non-Goals

- Rent, expiry, or a revoke opcode. Sending the output to an address you do not control is how a handle leaves you. The protocol does not burn it back to “available.”
- A start height that ignores older exact `@jon` inscriptions. That would be a premine.
- Verifying social accounts. That stays Ð𝕏 and ÐN05.
- Spaces, hyphens, or dotted names inside the handle. Hyphen belongs to DNS labels. Dot belongs to DNS names. A ZWJ sequence or a skin-tone modifier is not a character and is not a handle. A flag is one character.

---

## 5. Envelope

A handle exists only as the **body of an `ord` inscription**. Content type is `text/plain` or `application/json`. A `;charset=utf-8` parameter is accepted. Classification uses the body, not the type string: a body that is the JSON object below counts even when the type is `text/plain`.

The carrier is DNS §5.0, unchanged: P2SH commit, then one push-only reveal, envelope pushes in `vin[0].scriptSig`, inscription id `<reveal-txid>i0`. One reveal, one handle. The reference output value is 100000 koinu. Wallets SHOULD use the committed redeem so a relayer cannot swap the body. Broadcast commit, then reveal, through command.dog to Dogecoin Core. A public-relay txid that Core never returns is not a mint.

Worked plain-text registration of `@jon` (envelope pushes only):

```text
03 6f7264
51
0a 746578742f706c61696e
00
04 406a6f6e
```

That is `ord`, parts `1`, `text/plain`, countdown `0`, body `@jon` (`40 6a 6f 6e`).

Indexers MUST ignore:

| What was broadcast | Why it is not a handle |
| --- | --- |
| OP_RETURN, including `AT1\|reg\|@jon\|<address>` | No inscription body. |
| The JSON in §5.2 or §5.3 placed in an OP_RETURN | JSON counts only as inscription content. |
| `OP_FALSE OP_IF "ord" …` in a legacy scriptSig | Core rejects it. |
| A DNS body (`jon.doge`, or `p:"dns"`) | DNS. |
| A Ð𝕏 body (`p:"dx"`, or OP_RETURN `DX`) | Ð𝕏. |
| `op:"transfer"`, `op:"renew"`, `op:"revoke"` | Undefined here. Ignore. Revoke is a Ð𝕏 op, not a handle op. |

### 5.1 Plain text (normal mint)

UTF-8. Reject invalid UTF-8. Strip one leading U+FEFF. If the trimmed body starts with `{`, it is JSON (§5.2–§5.4), not plain text.

Otherwise remove one trailing `\n` or `\r\n` if present. The remainder MUST contain no whitespace. Fullwidth-fold (U+FF01..U+FF5E → the ASCII character 0x21..0x7E), ASCII-lowercase, strip U+FE0E and U+FE0F, then accept the body when it is a handle under §6. Stored form is `@jon`, `@jon🐕`, or `@🇺🇸`. `@Jon🐕` and `@jon🐕` plus a variation selector are `@jon🐕`.

Anything else is not a register. In particular, indexers MUST ignore:

- `jon` (no `@` — a bare word would swallow ordinary text inscriptions)
- `@jon is taken` (a second token)
- a paragraph that contains `@jon`
- `@jon.doge` (a dot)
- `@` alone, `@jon-heaven` (a hyphen), more than 15 characters
- a lone regional indicator, a ZWJ sequence, or a skin-tone modifier
- more than one line

The strict body is deliberate. DNS may ignore trailing tokens because `something.doge` rarely opens a sentence. `@` opens mentions constantly. A loose match would turn posts into squat inscriptions.

### 5.2 JSON register

```json
{
  "p": "at",
  "op": "register",
  "name": "@jon",
  "address": "D...",
  "url": "https://example.com",
  "site": "<inscription_id>",
  "avatar": "<inscription_id>",
  "content": "hello from chain",
  "cname": "@jon"
}
```

Required: `p` = `"at"`, `op` = `"register"` or `"reg"`, `name` a handle under §6. `name` MAY omit the leading `@` (`"jon"`). The stored handle still has it. All record fields are optional. Omit `address` unless the minter is setting a pay-to that can differ from the holder.

`rev` is an alias of `address`. `relay` is an alias of `site`. `website` is an alias of `url`. `insc` / `html` are aliases of `site`. A trailing comma before `}` is accepted.

`p` other than `"at"` is not this protocol, even if `name` looks like `@jon`.

A JSON register MAY carry `"bitworkc"` and `"bitworkr"` with the same meaning as DNS §5.7. A claim the txid does not meet voids the register.

### 5.3 JSON config

```json
{
  "p": "at",
  "op": "config",
  "name": "@jon",
  "url": "https://example.com",
  "content": "updated bio"
}
```

`op` MAY be `"update"` or `"upd"`. The config inscription is not the handle and does not transfer it.

Record fields MAY sit inside a `config` object. A top-level key wins when both are present. A body with `p:"at"`, a valid `name`, and record fields but no `op` is a config, not a second registration.

Omitted keys do not clear. `""` clears a scalar. `addresses: []` clears extra pay-tos. `txt: null` clears the TXT map. A TXT value of `""` deletes that key.

A config counts only when the handle already exists and the config inscription’s owner is the holder **after every spend in that same transaction has been applied**. A config that rides with a transfer does not stick to the new owner unless its output pays that owner too.

### 5.4 Primary

```json
{ "p": "at", "op": "primary", "name": "@jon" }
```

It counts only when `name` is already registered, the inscription owner is the current holder, and it is the latest such inscription from that holder. `{"p":"at","op":"primary","name":""}` clears it. Sending the handle away clears it for the previous holder. The new holder does not inherit it.

One address has at most one @ primary. That slot is not the DNS primary. An address may wear `@jon` and `jon.doge` at the same time.

### 5.5 Transfer

There is no transfer opcode. Spend the register output. The new owner is whoever can spend it now.

### 5.6 Priority

Registers rank by the DNS §5.7 priority key, with the same window `W = 30`. Lower key wins. A committed reveal whose commit confirmed at most 30 blocks earlier ranks by `(commit_height, commit_tx_index, commit_vout)`. Every other register ranks by `(reveal_height, reveal_tx_index, 0)`.

A free handle is taken by the first register. A taken handle is displaced only while the existing row is still pending (tip has not passed `existing_key_height + W`) and the new key is lower. A final row is never displaced. Later valid registers are missed claims.

Configs and primaries use block order: lower height, then lower transaction index, then lower inscription index. They do not use the commit-first key.

Reorgs undo creates and undo displacements, the same way DNS does.

There is no activation height. An exact `@jon` inscription from before this spec was published is a register if it matches §5.1 or §5.2.

---

## 6. Handle rules

Indexers MUST:

1. Fullwidth-fold, then ASCII-lowercase. Do not fold one emoji into another. `🐕` and `🐶` are different handles.
2. Take one leading `@`. On the JSON `name` field only, a missing `@` is inserted when the rest is a legal localpart. Plain text without `@` does not register.
3. Strip U+FE0E and U+FE0F. They are not characters and do not create a second handle.
4. Count characters, not bytes and not UTF-16 code units. Walk the remaining code points:
   - Two regional indicators (U+1F1E6 through U+1F1FF) are one character, a flag. A lone indicator rejects the handle.
   - Otherwise the code point is one character when it is `a-z`, `0-9`, `_`, or Extended_Pictographic in Unicode 18.0.0 `emoji-data.txt` (that property only).
5. The localpart is 1 to 15 of those characters, in any order. `@jon🐕` is 4. `@jon🇺🇸` is 4. Fifteen `🐕` is valid. Sixteen is not. `🇺🇸` is one character.
6. Reject a dot, a hyphen, a second `@`, whitespace, controls, any other script, U+200D, and an emoji modifier (U+1F3FB through U+1F3FF). A ZWJ sequence and a skin tone are not handles.

Underscore is allowed, including in front, because that is the handle shape people already type. DNS forbids underscore. That difference is another reason the two indexes stay split. `@jon` and `jon.doge` can be held by different people. Neither blocks the other.

Charset size for letters is 37 (26 letters, 10 digits, underscore) at each of 15 lengths. Emoji and flags share that same 15. Indexers MAY report how many of each length are taken. Length is the character count above, not the byte length. That number is not a price.

No handle is reserved. Not `jon`, not `satoshi`, not `doge`.

---

## 7. Records

Record fields use DNS §7’s validation, with two substitutions:

- `name` and `cname` are handles (§6), not `label.namespace`. A `cname` of `jon.doge` is ignored.
- A `lightning` value of `user@host` is a record. Indexers MUST NOT treat record values as registrations.

| Field | Meaning |
| --- | --- |
| `address` | Pay-to. Not the owner, and not the primary. |
| `url` | `http://` or `https://`, max 512 chars |
| `site` | Inscription id of on-chain content |
| `avatar` | Image inscription id or `https://` URL |
| `content` | Bio, max 280 UTF-8 chars |
| `cname` | Another handle |
| `nostr` | 64 hex or `npub1…` |
| `lightning` | `user@host` or `lnurl1…` |
| `addresses` | Up to 8 extra pay-tos |
| `txt` | Up to 8 keys, same limits as DNS |

The owner is always the inscription holder. `address` only changes where wallets send when someone types the handle.

---

## 8. Resolution against Ð𝕏

Wallets and pay pages MUST resolve the recipient `@jon` as follows:

1. Look up this index. If the handle is registered, the pay-to is the current holder’s `address` record when set, otherwise the owner of the register inscription. Show that Dogecoin address before the user approves.
2. If it is not registered, stop. Do not substitute a Ð𝕏 bind, a DNS name, or a ÐN05 name.
3. A Ð𝕏 lookup (`GET /api/dx/handle/jon`) MAY be offered beside it, labeled as the X-account claim, and only when the user picks that row.

Owning the inscription does not verify the X account. A verified Ð𝕏 bind does not reserve the inscription and does not move it. Product copy that says “you own @jon” means the inscription. Copy that says “X @jon” means Ð𝕏.

Until `/api/at/resolve` is live, a wallet that already pays to a Ð𝕏 bind MUST keep calling that destination the X bind. It MUST NOT label that payment as settling an on-chain handle.

---

## 9. Indexer state

Separate from DNS column families and from the Ð𝕏 bind map.

For each handle:

- canonical `handle`
- register `inscription_id`, height, timestamp, sequence
- priority key and `pending` | `final` (DNS §5.7)
- records after the winning register and later configs from the current holder
- `owner` from the register inscription’s current output
- missed claims
- primary, per address, for this protocol only

**FCFS** follows §5.6. A config inscription is not a second handle.

---

## 10. HTTP (dogex)

| Method | Path |
| --- | --- |
| GET | `/api/at/status` |
| GET | `/api/at/list?offset=&limit=&len=&q=&emoji=` |
| GET | `/api/at/recent?offset=&limit=` |
| GET | `/api/at/name/:handle` |
| GET | `/api/at/name/:handle/claims` |
| GET | `/api/at/address/:address` |
| GET | `/api/at/primary/:address` |
| GET | `/api/at/resolve/:handle` |

`:handle` accepts `@jon`, `jon`, and percent-encoded `%40jon`. Response `handle` is always the canonical form with `@`.

`resolve` is the wallet call. It returns `ok` or `available`. When `ok`, it includes `owner`, `payTo` (`address` record or owner), `inscriptionId`, and `status`. It does not include a Ð𝕏 address.

---

## 11. Product

**doginals.lol/handles** is the mint page. The body inscribed is the handle and nothing else. The page may mint before this index has replayed history. It MUST say that an earlier exact inscription can still win.

- `/handles` — claim one or more handles
- `/handles/@jon` — same page, handle filled in (`@` may be `%40`)
- Mint uses `signInscriptionTxs` (P2SH commit → reveal) and broadcasts through command.dog

`/names` stays DNS. A query with a dot is a domain. A handle is not a `.doge` search hit.

**Dig** shows held handles next to `.doge` names once `/api/at/resolve` has history. Until that replay, a wallet that already pays a Ð𝕏 bind keeps calling that destination the X bind.

A marketplace lists the register inscription the way it lists any Doginal. It MUST show the canonical handle from this index. A missed-claim inscription MUST be labeled as a duplicate, not as the handle. There is no separate handle order type.

---

## 12. Security

- FCFS is economic. There is no UDRP and no staff undo.
- A classic reveal can be copied out of the mempool. Use a committed reveal and let the commit confirm (§5.6).
- The classic redeem lets a relayer swap pushes. The committed redeem does not.
- Whoever can spend the output owns the handle.
- `url` can phish. A gateway that redirects MUST show the destination first.
- This spec does not check X, so a squatter can hold `@jon` while someone else has the X account. Wallets MUST NOT merge those two facts into one checkmark.

---

## 13. Examples

See [`examples/`](examples/) and [`vectors/conformance.json`](vectors/conformance.json).

Plain mint:

```text
@jon
```

JSON mint:

```json
{"p":"at","op":"reg","name":"jon"}
```

That JSON registers `@jon`. It does not register `jon.doge`.

Not a mint:

```text
AT1|reg|@jon|D8mZsgKwmSQWYRPEMCcm8KzNFj1JVF5UpA
```

---

## 14. Conformance

An implementation is conformant when:

1. The plain-text and JSON vectors in `vectors/conformance.json` parse as specified.
2. A DNS body and a Ð𝕏 body produce no @ row.
3. Two registers of `@jon` leave one owner, and the other is a missed claim, ordered by §5.6.
4. Spending the register output changes the owner without a protocol opcode.
5. `/api/at/resolve/@jon` never returns a Ð𝕏 pay-to.
