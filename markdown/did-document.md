## DID Document

A conformant `did:cid` resolution returns only the three members defined by the [[ref: DID-CORE]] DID Resolution data model — the *resolution result*:

```json
{
  "didDocument": { ... },
  "didResolutionMetadata": {
    "contentType": "application/did+ld+json"
  },
  "didDocumentMetadata": { ... }
}
```

`didDocument` and `didDocumentMetadata` conform to [[ref: DID-CORE]]; `didResolutionMetadata` conforms to the DID Resolution specification. On the conformant surface `didResolutionMetadata` carries `contentType` (the media type of the returned representation — see the Representations subsection under DID Resolution); it does **not** carry the `retrieved` timestamp returned by the legacy endpoint.

The method-specific `didDocumentData` and `didDocumentRegistration` objects are **not** part of the resolution result. They are Archon extensions (described in the Archon Extensions to DID Core section), exposed as dereferenceable resources at the `/data` and `/registration` DID URLs (see DID Resolution and DID URL Dereferencing). Together with the resolution members they form the internal *document set*:

```json
{
  "didDocument": { ... },
  "didDocumentMetadata": { ... },
  "didDocumentData": { ... },
  "didDocumentRegistration": { ... },
  "didResolutionMetadata": {
    "retrieved": "2026-06-15T19:22:45.691Z"
  }
}
```

The legacy `/api/v1/did/<did>` endpoint returns this full document set inline for backwards compatibility. The examples in this section show the relevant document-set members for each DID type; under the conformant surface, `didDocumentData` and `didDocumentRegistration` are retrieved by dereferencing rather than inline.

::: note
The [[ref: operation chain]] is the authoritative source of truth for a `did:cid` DID. The Gatekeeper stores individual operations (create, update, delete) and reconstructs the DID document by replaying them in [[ref: ordinal key]] order at resolution time. Implementations MAY cache resolved documents for performance, but any cached result MUST remain consistent with a fresh replay of the canonical operation chain. The `didResolutionMetadata.retrieved` timestamp shown in the document set above records when the resolution response was generated; it is returned only by the legacy `/api/v1/did/<did>` endpoint. The conformant surface returns `contentType` instead.
:::

---

### Agent DID Document

A resolved [[ref: agent]] DID document includes a verification method and the standard DID Core verification relationships:

```json
{
  "didDocument": {
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:cid:bafkreig6rjxbv2aopv47dgxhnxepqpb4yrxf2nvzrhmhdqthojfdxuxjbe",
    "verificationMethod": [
      {
        "id": "#key-1",
        "controller": "did:cid:bafkreig6rjxbv2aopv47dgxhnxepqpb4yrxf2nvzrhmhdqthojfdxuxjbe",
        "type": "EcdsaSecp256k1VerificationKey2019",
        "publicKeyJwk": {
          "kty": "EC",
          "crv": "secp256k1",
          "x": "LRrQabMIkvGVTA2IRk0JdWCpu57MNGm89nugrBZHo24",
          "y": "KHsWAaidAIGCosDjRYDIk-94793e4xVEL4UwFxjWgB8"
        }
      }
    ],
    "authentication": ["#key-1"],
    "assertionMethod": ["#key-1"]
  },
  "didDocumentMetadata": {
    "created": "2026-01-14T19:29:06Z",
    "versionId": "bafkreig6rjxbv2aopv47dgxhnxepqpb4yrxf2nvzrhmhdqthojfdxuxjbe",
    "versionSequence": "1"
  },
  "didDocumentData": {},
  "didDocumentRegistration": {
    "version": 1,
    "type": "agent",
    "registry": "hyperswarm"
  }
}
```

#### Verification Relationships

`did:cid` agent DIDs support the following [[ref: DID-CORE]] verification relationships:

| Relationship | Purpose |
|---|---|
| `authentication` | Proves control of the DID — used when the agent must authenticate itself to a verifier |
| `assertionMethod` | Signs verifiable credentials and other assertions |

The initial verification method is referenced as `#key-1`. After key rotation via an update operation, the new method identifier increments (`#key-2`, `#key-3`, etc.) and both `authentication` and `assertionMethod` are updated to reference the new key. Historical keys remain resolvable via [[ref: temporal resolution]].

#### Authentication

An [[ref: agent]] authenticates by signing a challenge with the private key corresponding to the method in its `authentication` verification relationship. A verifier resolves the agent's DID (at the time of the challenge) to obtain the then-active public key, then verifies the signature.

This enables DID-native, decentralized authentication: any party that can resolve a `did:cid` DID can authenticate a `did:cid` agent without relying on a centralized identity provider or certificate authority.

#### Service Endpoints

Agent DIDs may include service endpoint entries per [[ref: DID-CORE]] Section 5.4. Services are declared in the `service` array and updated via standard DID update operations:

```json
{
  "service": [
    {
      "id": "#lightning",
      "type": "lightning",
      "serviceEndpoint": "https://drawbridge.example.com/lightning"
    }
  ]
}
```

Service endpoints are optional. Any DID Core-conformant service type may be used; `did:cid` imposes no constraints on service endpoint structure beyond those defined in [[ref: DID-CORE]].

---

### Asset DID Document

A resolved [[ref: asset]] DID document identifies its controlling agent and carries application data in `didDocumentData`. It has no verification methods of its own:

```json
{
  "didDocument": {
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:cid:bagaaiera...asset",
    "controller": "did:cid:bagaaieradidcs4hohalzexldr5mdmbmt553tqq3ifqd56mvhifppvyfdc32q"
  },
  "didDocumentMetadata": {
    "created": "2026-01-14T19:32:24Z",
    "versionId": "bagaaiera...asset",
    "versionSequence": "1"
  },
  "didDocumentData": {
    "group": {
      "name": "testgroup",
      "members": []
    }
  },
  "didDocumentRegistration": {
    "version": 1,
    "type": "asset",
    "registry": "hyperswarm"
  }
}
```

Asset DIDs support transfer of control: a controller may update the `controller` field to a new agent DID via a valid update operation. Only one controller is valid at any given time.

---

### Metadata Objects

#### didResolutionMetadata

The `didResolutionMetadata` object is added by the Gatekeeper at resolution time and conforms to the DID Resolution specification. The field returned depends on the surface:

| Field | Surface | Description |
|-------|---------|-------------|
| `contentType` | Conformant | Media type of the returned representation (`application/did+ld+json` or `application/did+json` — see the Representations subsection under DID Resolution) |
| `retrieved` | Legacy | ISO 8601 timestamp of when this resolution was computed |

The conformant `/1.0/identifiers` surface returns `contentType`; the legacy `/api/v1/did/<did>` endpoint returns `retrieved`. Because `retrieved` is set fresh on every call, no two legacy resolution responses for the same DID are identical — even when the underlying DID document has not changed.

#### didDocumentMetadata

The `didDocumentMetadata` object conforms to [[ref: DID-CORE]] and includes Archon-specific extensions:

| Field | Source | Description |
|-------|--------|-------------|
| `created` | DID Core | ISO 8601 timestamp of initial DID creation |
| `updated` | DID Core | ISO 8601 timestamp of most recent update |
| `deactivated` | DID Core | `true` if the DID has been revoked via a delete operation |
| `versionId` | DID Core | CID of the most recent operation in the [[ref: operation chain]] |
| `versionSequence` | Archon | Sequence number of the most recent operation, returned as a string |
| `confirmed` | Archon | `true` if the most recent operation is confirmed on the [[ref: registry]] |
| `timestamp` | Archon | Blockchain timestamp bounds (blockchain registries only — see below) |

::: note
`confirmed` and `timestamp` are method-specific anchoring provenance, not [[ref: DID-CORE]] document metadata. They appear inline in `didDocumentMetadata` only on the legacy `/api/v1/did/<did>` endpoint. The conformant surface strips both from `didDocumentMetadata` and returns them with the `/registration` resource instead (see DID URL Dereferencing).
:::

#### Blockchain Timestamp Bounds

For DIDs using blockchain-based registries (Bitcoin, Ethereum, Zcash, Solana, Filecoin), the `timestamp` object provides cryptographic upper and lower bounds on when the most recent operation was submitted, derived directly from block data:

```json
{
  "didDocumentMetadata": {
    "versionId": "bafkrei...",
    "versionSequence": "2",
    "confirmed": true,
    "timestamp": {
      "chain": "BTC",
      "lowerBound": {
        "time": 1705312800,
        "timeISO": "2024-01-15T10:00:00Z",
        "blockid": "00000000000000000002a7c4...",
        "height": 826000
      },
      "upperBound": {
        "time": 1705316400,
        "timeISO": "2024-01-15T11:00:00Z",
        "blockid": "00000000000000000001b8f2...",
        "height": 826005,
        "txid": "a1b2c3d4e5f6...",
        "txidx": 42,
        "batchid": "bafkrei...",
        "opidx": 3
      }
    }
  }
}
```

**Lower bound** (`lowerBound`): Present when the operation included a `blockid` field at submission time, referencing a recent block. This proves the operation was created *after* that block was mined — establishing a cryptographic "not before" constraint.

**Upper bound** (`upperBound`): Always present for confirmed blockchain operations. Identifies the block in which the operation batch was anchored, proving the operation existed *before* the subsequent block — establishing a "not after" constraint.

Together, the bounds define an independently verifiable time window without relying on self-asserted client timestamps. The bounds can be verified by any party with access to the relevant blockchain, providing legal-grade timestamping for DID operations.

::: note
For [[ref: registry, registries]] without blockchain consensus (e.g., Hyperswarm), the `timestamp` object is absent. Operation ordering on such registries relies on the P2P consensus mechanism of the registry itself rather than external block timestamps.
:::
