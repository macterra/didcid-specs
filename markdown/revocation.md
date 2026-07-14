## DID Revocation

[[def: delete operation, A signed operation that permanently deactivates a DID by removing its controller, making the DID unresolvable for active use]]

Revoking a DID is a special kind of Update that results in the termination of the DID. Revoked DIDs cannot be updated because they have no controller, therefore they **cannot be recovered** once revoked. Revoked DIDs can be resolved without error, but resolvers will return a result with the `didDocumentMetadata.deactivated` property set to `true`. The `didDocument` is reduced to just its `id`, and the DID's data resource (dereferenced at `/data`) is empty.

### Revocation Flow

To revoke a DID, the client must sign and submit a `delete` operation to a node:

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Must be `"delete"` |
| `did` | Yes | The DID to be deleted |
| `previd` | Yes | The CID of the previous operation |
| `blockid` | No | Current block ID on registry (if blockchain) |

### Delete Operation Example

```json
{
    "type": "delete",
    "did": "did:cid:bagaaiera7vfnrxrmcvo7prrbmdhpvusroii4y2gir252nzk4jv5nxgkzldha",
    "previd": "bagaaiera7vfnrxrmcvo7prrbmdhpvusroii4y2gir252nzk4jv5nxgkzldha",
    "proof": {
        "type": "EcdsaSecp256k1Signature2019",
        "created": "2026-01-14T19:34:32.170Z",
        "verificationMethod": "did:cid:bagaaieradidcs4hohalzexldr5mdmbmt553tqq3ifqd56mvhifppvyfdc32q#key-1",
        "proofPurpose": "authentication",
        "proofValue": "YUTouPmhHDSudPSJ9iU44HdzBYDm7cqmDmanhgDLa4A3MBNiJpbWL2Db4BbzDYQ4NjJCRDWixYZOT2ojzzBHI3c"
    }
}
```

Upon receiving the operation, the node must:

1. Verify the proof is valid for the controller of the DID.
1. Verify the `previd` is identical to the latest version's operation CID.
1. Record the operation on the DID's specified registry (or forward the request to a trusted node that supports the specified registry).

### Post-Revocation Resolution

After revocation is confirmed on the DID's registry, resolving the DID returns a result like this:

```json
{
    "didDocument": {
        "id": "did:cid:bagaaiera7vfnrxrmcvo7prrbmdhpvusroii4y2gir252nzk4jv5nxgkzldha"
    },
    "didResolutionMetadata": {
        "retrieved": "2026-01-14T19:36:09.115Z"
    },
    "didDocumentMetadata": {
        "deactivated": true,
        "created": "2026-01-14T19:32:24Z",
        "deleted": "2026-01-14T19:34:33Z",
        "versionId": "bagaaierats6ttxvpx2l3tat25ota7z7335akfd2iup5loajsdlqcwismkgpq",
        "versionSequence": "2",
        "confirmed": true
    }
}
```

The metadata `deactivated` field is set to `true` to conform to the [[ref: DID-CORE]] specification for [DID Document Metadata](https://www.w3.org/TR/did-core/#did-document-metadata). The revoked DID's data resource, dereferenced at `/data`, is empty.

::: warning
Revocation is **irreversible**. Once a DID is deactivated, there is no controller to sign a recovery operation. Ensure all credentials and references have been migrated before revoking a DID.
:::