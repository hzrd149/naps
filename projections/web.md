Web Projection — NIP-5D
=======================

The **web projection** of the NAP capability seam. It maps the binding-neutral
contracts in the [registry](../README.md) onto the browser.
[NIP-5D](https://github.com/nostr-protocol/nips/pull/2303) is the legacy
upstream reference for iframe loading, sandboxing, namespace injection, and
transport. The [web napplet event](../WEB-NAPPLET.md) defines the artifact and
identity that this projection loads.

A *projection* answers four binding-specific questions for one host environment:
where napplets run, how messages travel, how a napplet's identity is bound, and
how each NAP **domain** is surfaced. The contracts themselves (operations,
schemas, error models, trust boundaries) do not change between projections.

## At a glance

| Concern | Web projection |
|---------|----------------|
| Host | Napplets run as `sandbox="allow-scripts"` iframes |
| Carrier | Messages travel over `postMessage` |
| Surface | Capabilities and convention URI transposition appear on `window.napplet.*` |
| Discovery | `shell.supports("<domain>")` |
| Identity | Runtime verifies `MessageEvent.source` and binds each message to a napplet |

## Domain surfacing

A NAP is named in the registry by its **domain** (`relay`, `intent`, …). In the
web projection, domain `X`:

- surfaces as the object `window.napplet.X`, and
- is discovered via `shell.supports("X")`.

So `NAP-RELAY` (domain `relay`) is reached at `window.napplet.relay` and probed
with `shell.supports("relay")`. Other projections map the same domains into their
own host idiom.

## Message delivery

Request/result objects (the `domain.action` envelopes described in the registry)
are delivered by `postMessage`:

```
-> { "type": "relay.publish", "id": "a1", "event": { … } }   // napplet → shell
<- { "type": "relay.publish.result", "id": "a1", "ok": true } // shell → napplet
```

## Convention URI binding

A developer MAY pass
`napplet:<archetype>/<intent>[...?params]` to a `window.napplet.*` operation that
accepts a convention URI. Before `postMessage`, the web binding:

1. removes the query from the stable convention identity,
2. percent-decodes each unique `name=value` pair as text, and
3. places those pairs in the operation's payload object.

The binding MUST NOT coerce scalar types or apply form-encoding semantics: `+`
is a literal plus sign. It MUST reject fragments, malformed percent-encoding,
repeated names, and a query combined with an explicit payload before sending a
message. The shell receives normalized identity and payload fields. Routing and
handler resolution use exact equality over the queryless identity.

## Identity & trust

The shell is the policy boundary. For every inbound message it verifies
`MessageEvent.source` to bind the message to a napplet identity — the
`(35129:<pubkey>:<d>, artifactHash)` tuple, assigned after the shell verifies the
signed [web napplet event](../WEB-NAPPLET.md) and its HTML artifact, not
negotiated by the napplet. Napplets are untrusted: they never receive signing
keys, wallet credentials, or raw network access. Security-critical operations
are performed by the shell on the napplet's behalf, gated by per-napplet
capability policy.

## References

- [Web napplet event](../WEB-NAPPLET.md) — kind `35129`, artifact, and identity
- [NIP-5D](https://github.com/nostr-protocol/nips/pull/2303) — legacy upstream web loading and transport reference
- [Registry & governance](../README.md)
