# Implement a Dogecoin @ handle mint

Read this, then [spec.md](spec.md). If they disagree, spec.md wins.

## The mint

Content type `text/plain` (indexers ignore the type; the body decides). The body is the handle and nothing else:

```text
@jon
```

That is the whole inscription. One optional trailing newline is accepted. A second word, a sentence that mentions `@jon`, or `jon` with no `@` does not register. A handle is 1 to 15 characters. A character is a letter, digit, underscore, one Extended_Pictographic scalar, or a flag (two regional indicators, counted as one). `@jon🐕` and `@🇺🇸` register. A lone regional indicator, a ZWJ sequence, and a skin tone do not.

The envelope is the DNS carrier: P2SH commit, then a push-only reveal, `ord` pushes in `vin[0].scriptSig`. Same steps as [DNS](https://github.com/jonheaven/dns) (spec §5.0). Do not invent a second envelope. Do not put the handle in an OP_RETURN.

```text
AT1|reg|@jon|D8mZsgKwmSQWYRPEMCcm8KzNFj1JVF5UpA
```

That nulldata string is not a handle. Same failure mode as `DNS1|reg|…`.

## Records

A later inscription from the current holder, content type `application/json`:

```json
{"p":"at","op":"config","name":"@jon","address":"D...","content":"hello"}
```

`p` is the two letters `at`. `p:"@"` is not a marker. `p:"dns"` and `p:"dx"` are other protocols and do not create or update a handle.

Records belong to the holder. Sell or move the handle to another address and they reset; the new holder inscribes their own config.

## Before you broadcast `@jon`

Rules apply to every earlier exact inscription. There is no start height that ignores history. dogex lane `at-handles` scans from block 4,600,000 into the `at_*` column families and is on unless `DOGEX_INDEX_AT_SIDELANE=0`. It skips transactions that are not inscriptions. Until `GET /api/sidelanes/at-handles` is caught up, a missing row means "not scanned yet". A historical inscription whose body is exactly `@jon` already won.

## What wallets resolve

The recipient string `@jon` means this handle: the holder’s `address` record if they set one, otherwise the inscription owner. A Ð𝕏 bind for the X account `@jon` is a different lookup. Show it only as an explicit “X bind” choice. Do not send there because the user typed `@jon`.

## Indexer outline (dogex)

Keep a separate column family from `dns_namespace` and from the Ð𝕏 bind index.

1. Parse inscription bodies with the rules in spec §5–§6.
2. Rank registers with the DNS §5.7 priority key (`W = 30`) everywhere — the tip indexer and the `at-handles` replay lane alike. The lane keeps the last 30 blocks of P2SH outputs so a committed reveal ranks by its commit, and it voids a JSON register whose `bitworkc`/`bitworkr` claim fails. Ranking a replay by reveal height alone disagrees with a live indexer (spec §5.6).
3. Owner is the current holder of the register inscription output. No `transfer` opcode.
4. Apply config and primary only after spends in that transaction, and only from the holder. Same timing as DNS.
4b. When the register output moves to a different owner, reset every record. A move to the same address keeps them (spec §5.3).
5. Serve `/api/at/status`, `/list`, `/recent`, `/stats`, `/name/:handle`, `/name/:handle/claims`, `/name/:handle/history`, `/address/:address`, `/primary/:address`, `/resolve/:handle`. `resolve` returns `payTo: null` for a burned handle.

Product: [doginals.lol/handles](https://doginals.lol/handles). Dig keeps paying a Ð𝕏 bind as an X bind until `/api/at` has history.
