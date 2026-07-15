## DID URL Dereferencing

[[def: dereferencing, The process of returning a resource identified by a DID URL — for did:cid, a path resource or a node within the DID document — as distinct from resolution, which returns the DID document and its metadata]]

Dereferencing a `did:cid` DID URL returns a *resource* associated with the DID, as distinct from [[ref: resolution]] (which returns the DID document and its metadata). Because `did:cid` is [[ref: content-addressed identity, content-addressed]], these resources are retrieved by content rather than from an external location.

### Path Resources

This method defines two dereferenceable resources, selected by the DID URL path:

- **`/data`** — `did:cid:<cid>/data` dereferences to the DID's data resource: the bare [[ref: didDocumentData]] object from the internal document set. [[ref: agent, Agent]] DIDs have an empty data resource (`{}`); [[ref: asset]] DIDs return their attached data. A revoked DID's data resource is empty.
- **`/registration`** — `did:cid:<cid>/registration` dereferences to the DID's registration/anchoring provenance: the [[ref: didDocumentRegistration]] object (registry, type, validity, version), **plus** the anchoring state `confirmed` and, where the registry anchors to a blockchain, `timestamp`. This is method-specific provenance, not [[ref: DID-CORE]] DID document metadata, which is why it is dereferenced rather than embedded in `didDocumentMetadata`. This is where `confirmed` and `timestamp` are surfaced on the conformant surface, having been stripped from `didDocumentMetadata`.

```json
{
  "registry": "BTC:mainnet",
  "type": "agent",
  "version": 1,
  "confirmed": true,
  "timestamp": {
    "chain": "BTC:mainnet",
    "opid": "bagaaiera...",
    "lowerBound": { "time": 1768424000, "timeISO": "2026-01-14T19:33:20Z", "blockid": "0000...", "height": 878123 },
    "upperBound": { "time": 1768425200, "timeISO": "2026-01-14T19:53:20Z", "blockid": "0000...", "height": 878125, "txid": "…", "txidx": 3 }
  }
}
```

`timestamp` is present only when the DID's registry anchors to a blockchain and the anchoring block is known; registries such as `hyperswarm` return the registration fields and `confirmed` alone.

Neither resource is part of the DID resolution result. Both honor the `versionTime` and `versionSequence` version selectors, and both are returned as plain `application/json`.

::: note
Under the conformant `/1.0/identifiers` surface these resources always reflect confirmed, cryptographically verified state. The legacy `/api/v1/did/<did>` endpoint continues to return `didDocumentData` and `didDocumentRegistration` inline within the full document set (with `confirmed` and `timestamp` carried in `didDocumentMetadata`).
:::

### Fragments

A fragment identifies a node within the DID document — for example `did:cid:<cid>#key-1` dereferences to the verification method whose `id` matches. Per the [[ref: DID-CORE]] processing model, the fragment is applied **client-side** to the resolved DID document (it is not transmitted to the resolver over HTTP); the node whose fully-qualified `id` matches the DID URL is returned.

### Service Dereferencing (Future)

The registered `service` and `relativeRef` query parameters — used to construct external service-endpoint URLs — are not currently implemented. If added they would be additive, and would change neither the resolution result nor the path resources above.
