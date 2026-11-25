# Implement a Dogecoin @ handle mint

Read this, then [spec.md](spec.md). If they disagree, spec.md wins.

## The mint

Content type `text/plain`. The body is the handle and nothing else:

```text
@jon
```

That is the whole inscription. One optional trailing newline is accepted. A second word, a sentence that mentions `@jon`, or `jon` with no `@` does not register.

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

`p` is the two letters `at`. `p:"dns"` and `p:"dx"` are other protocols and do not create or update a handle.

## Before you broadcast `@jon`

Rules apply to every earlier exact inscription. There is no start height that ignores history. dogex lane `at-handles` scans from block 4,600,000 into the `at_*` column families and is on unless `DOGEX_INDEX_AT_SIDELANE=0`. It skips transactions that are not inscriptions. Until `GET /api/sidelanes/at-handles` is caught up, a missing row means "not scanned yet". A historical inscription whose body is exactly `@jon` already won.

## What wallets resolve

The recipient string `@jon` means this handle: the holder’s `address` record if they set one, otherwise the inscription owner. A Ð𝕏 bind for the X account `@jon` is a different lookup. Show it only as an explicit “X bind” choice. Do not send there because the user typed `@jon`.

## Indexer outline (dogex)

Keep a separate column family from `dns_namespace` and from the Ð𝕏 bind index.

1. Parse inscription bodies with the rules in spec §5–§6.
2. The tip indexer ranks registers with the DNS §5.7 priority key (`W = 30`). The `at-handles` lane ranks by reveal height so it can scan from the first Doginal block without opening every transaction. A lower height replaces a later row.
3. Owner is the current holder of the register inscription output. No `transfer` opcode.
4. Apply config and primary only after spends in that transaction, and only from the holder. Same timing as DNS.
5. Serve `/api/at/status`, `/list`, `/name/:handle`, `/name/:handle/claims`, `/address/:address`, `/primary/:address`, `/resolve/:handle`.

Product: [doginals.lol/handles](https://doginals.lol/handles). Dig keeps paying a Ð𝕏 bind as an X bind until `/api/at` has history.
