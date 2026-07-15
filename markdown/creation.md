## DID Creation

[[def: create operation, The initial operation that anchors a new DID to IPFS, producing the CID that becomes the DID suffix]]

DIDs are anchored to IPFS prior to any declaration on a registry. This allows DIDs to be created very quickly (less than 10 seconds) and at (virtually) no cost.

The `did:cid` method supports two main types of DID Subject: [[ref: agent]] and [[ref: asset]]. Agents have keys and control assets. Assets do not have keys, and are controlled by a single agent (the owner of the asset). The two types have slightly different creation methods.

### Create an Agent DID

To create an [[ref: agent]] DID, the client must sign and submit a create operation to a node:

1. Generate a new private key
   - We recommend deriving a new private key from a Hierarchical Deterministic (HD) wallet (BIP-32).
1. Generate a public key from the private key.
1. Convert the public key to [[def: JWK, JSON Web Key — a JSON representation of a cryptographic key, per RFC 7517]] format.
1. Create an operation object with these fields in any order:

   | Field | Required | Description |
   |-------|----------|-------------|
   | `type` | Yes | Must be `"create"` |
   | `registration.version` | Yes | Version number, e.g. `1` |
   | `registration.type` | Yes | Must be `"agent"` |
   | `registration.registry` | Yes | A valid registry identifier, e.g. `"BTC"`, `"hyperswarm"` |
   | `publicJwk` | Yes | The public key in JWK format |
   | `created` | Yes | ISO 8601 timestamp |
   | `blockid` | No | Current block ID on registry (if registry is a blockchain) |

1. Sign the JSON with the private key corresponding to the public key (this enables the node to verify that the operation is coming from the owner of the public key).
   - The `proof.verificationMethod` must be set to `#key-1` (a relative reference) since the DID does not yet exist.
1. Submit the signed operation to a node's DID endpoint.

#### Agent Create Example

```json
{
    "type": "create",
    "created": "2026-01-14T19:29:06.924Z",
    "registration": {
        "version": 1,
        "type": "agent",
        "registry": "hyperswarm"
    },
    "publicJwk": {
        "kty": "EC",
        "crv": "secp256k1",
        "x": "LRrQabMIkvGVTA2IRk0JdWCpu57MNGm89nugrBZHo24",
        "y": "KHsWAaidAIGCosDjRYDIk-94793e4xVEL4UwFxjWgB8"
    },
    "proof": {
        "type": "EcdsaSecp256k1Signature2019",
        "created": "2026-01-14T19:29:06.927Z",
        "verificationMethod": "#key-1",
        "proofPurpose": "authentication",
        "proofValue": "qNT0EhtojDxOJBh71pddmWnMharQZJxOelW71ehFfuZqrqPls32zSP4bD2CyYNEvAXSJRA-3X5DwR1vHVyTPHw"
    }
}
```

Upon receiving the operation, the node must:

1. Verify the proof.
1. Apply [[ref: JCS]] to the operation object.
1. Pin the [[ref: seed document]] to IPFS.

The resulting content address (CID) in standard CID v1 base32 encoding is used as the DID suffix. For example the operation above corresponds to CID `bafkreig6rjxbv2aopv47dgxhnxepqpb4yrxf2nvzrhmhdqthojfdxuxjbe`, yielding the DID:

`did:cid:bafkreig6rjxbv2aopv47dgxhnxepqpb4yrxf2nvzrhmhdqthojfdxuxjbe`

---

### Create an Asset DID

To create an [[ref: asset]] DID, the client must sign and submit a create operation to a node. Unlike an agent, an asset does not possess its own keys — it is controlled by an existing agent.

1. Create an operation object with these fields in any order:

   | Field | Required | Description |
   |-------|----------|-------------|
   | `type` | Yes | Must be `"create"` |
   | `registration.version` | Yes | Version number, e.g. `1` |
   | `registration.type` | Yes | Must be `"asset"` |
   | `registration.registry` | Yes | A valid registry identifier |
   | `controller` | Yes | The DID of the owner/controller agent |
   | `data` | Yes | Any non-empty JSON data payload |
   | `created` | Yes | ISO 8601 timestamp |
   | `blockid` | No | Current block ID on registry (if blockchain) |

1. Sign the JSON with the private key of the controller.
   - The `proof.verificationMethod` must be the **full DID reference** of the controller (e.g., `did:cid:abc123#key-1`).
1. Submit the signed operation to a node's DID endpoint.

#### Asset Create Example

```json
{
    "type": "create",
    "created": "2026-01-14T19:32:24.354Z",
    "registration": {
        "version": 1,
        "type": "asset",
        "registry": "hyperswarm"
    },
    "controller": "did:cid:bagaaieradidcs4hohalzexldr5mdmbmt553tqq3ifqd56mvhifppvyfdc32q",
    "data": {
        "group": {
            "name": "testgroup",
            "members": []
        }
    },
    "proof": {
        "type": "EcdsaSecp256k1Signature2019",
        "created": "2026-01-14T19:32:24.375Z",
        "verificationMethod": "did:cid:bagaaieradidcs4hohalzexldr5mdmbmt553tqq3ifqd56mvhifppvyfdc32q#key-1",
        "proofPurpose": "authentication",
        "proofValue": "NGQMBq5venJ2i4F3-Uo0p_rEAlY0zr-YJeTTu7vUlZ0NfyqirIPISGGyy8KU-QrBvsCrfc0fsQm8sh-2BfAzqQ"
    }
}
```

Upon receiving the operation, the node must:

1. Verify the proof is valid for the specified controller.
1. Apply [[ref: JCS]] to the operation object.
1. Pin the seed document to IPFS.