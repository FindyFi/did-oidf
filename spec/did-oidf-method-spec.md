# DID Method Specification: did:oidf

Authors:
- Samuel Rinnetmäki, Findynet Cooperative <samuel.rinnetmaki@findy.fi>

## Abstract

This document defines a Decentralized Identifier (DID) method that leverages OpenID Federation Entity Configuration files as DID Documents. This method, identified as `did:oidf`, provides a bridge between the OpenID Federation trust framework and the decentralized identity ecosystem using DIDs.

## Status of This Document

This document is a draft of the `did:oidf` DID method specification.

## Introduction

Decentralized Identifiers (DIDs) are identifiers for verifiable, self-sovereign digital identity. DIDs are fully under the control of the DID subject, independent from any centralized registry, identity provider, or certificate authority.

The OpenID Federation 1.0 specification defines Entity Configuration files which contain metadata about entities participating in an OpenID Federation. These Entity Configuration files provide cryptographic verifiability similar to DID Documents.

This specification defines the `did:oidf` DID method, which uses OpenID Federation Entity Configuration files as DID Documents, allowing entities participating in OpenID Federation to become part of the DID ecosystem.

## Method Name

The method name that identifies this DID method is: `oidf`.

A DID that uses this method MUST begin with the following prefix: `did:oidf:`. Per the DID specification, this prefix MUST be in lowercase.

## Method Specific Identifier

The method specific identifier is represented as an entity URL where the entity's OpenID Federation Entity Configuration is hosted, encoded according to the rules below.

The format of the `did:oidf` method specific identifier is:

```
did-oidf-format := did:oidf:<entity>
entity := url-encoded-entity-authority
```

Where:
- `entity` is the URL-encoded domain authority (host:port) of the OpenID Federation entity, without the scheme and path components

For example, an OpenID Federation entity with an Entity Configuration at `https://example.com/.well-known/openid-federation` would have the DID:

```
did:oidf:example.com
```

If the Entity Configuration is located at a non-standard port, that should be included:

```
did:oidf:example.com%3A8443
```

## CRUD Operations

### Create (Register)

To create a `did:oidf` DID:

1. The controller creates a key pair (or multiple key pairs) for signing.
2. The controller creates an OpenID Federation Entity Configuration in accordance with the OpenID Federation 1.0 specification, including the appropriate cryptographic materials.
3. The controller hosts this Entity Configuration at the standard well-known location (`/.well-known/openid-federation`) on their authority domain.
4. The DID is now active and can be resolved.

### Read (Resolve)

To resolve a `did:oidf` DID to its DID Document:

1. Parse the DID to extract the `entity` component.
2. URL-decode the `entity` component.
3. Construct the well-known URL: `https://{entity}/.well-known/openid-federation`
4. Retrieve the Entity Configuration from this URL.
5. Transform the Entity Configuration into the DID Document data model according to the transformation rules defined in Section 5.

If the Entity Configuration cannot be retrieved or does not conform to the OpenID Federation 1.0 specification, the resolution MUST fail.

### Update

To update a `did:oidf` DID Document:

1. The controller updates their OpenID Federation Entity Configuration.
2. The controller continues to host this updated configuration at the same well-known location.

Updates to the Entity Configuration MUST maintain validity per the OpenID Federation 1.0 specification, including proper key management and signing.

### Deactivate (Revoke)

To deactivate a `did:oidf` DID:

1. The conroller SHOULD set the `exp` claim of the Entity Configuration to reflect the deactivation time.
2. Alternatively, the controller MAY remove the Entity Configuration from the well-known location.

## Entity Configuration to DID Document Transformation

The OpenID Federation Entity Configuration must be transformed into the DID Document data model when resolving a `did:oidf` DID. This section defines the mapping between Entity Configuration elements and DID Document data model.

### Basic Properties

| Entity Configuration                  | DID Document                      | Transformation                                                                                  |
|---------------------------------------|-----------------------------------|-------------------------------------------------------------------------------------------------|
| (location)                            | `id`                              | The full DID URI (`did:oidf:{entity}`)                                                          |
| `sub`                                 | `alsoKnownAs`                     | none                                                                                            |
| `iat`                                 | `created`                         | Convert Unix timestamp to ISO8601 format                                                        |
| `exp`                                 | `updated`                         | Convert Unix timestamp to ISO8601 format                                                        |

### Verification Methods

The `jwks` property in the Entity Configuration maps to the `verificationMethod` property in the DID Document. Each JWK in the `jwks` array is transformed into a verification method entry as follows:

1. Generate a unique identifier for each key by combining the DID with a fragment that includes the key's `kid` value.
2. Map the JWK parameters to the appropriate verification method properties based on the key type.
3. Set the appropriate relationship between the DID subject and the verification method. 

### Services

If the Entity Configuration contains service endpoints (such as in the `metadata` section), these are mapped to the `service` property in the DID Document. Each service endpoint is given a unique identifier and mapped according to its type.

### Additional Properties

Any additional properties in the Entity Configuration that are relevant to the DID ecosystem but do not have a direct mapping to standard DID Document properties MAY be included in a `property` object in the DID Document.

## Security Considerations

### Key Management

The security of a `did:oidf` DID depends on the security of the private keys associated with the verification methods listed in the DID Document. Controllers MUST implement appropriate key management practices.

### Transport Security

Resolution of `did:oidf` DIDs involves retrieving Entity Configuration files over HTTPS. Implementations MUST validate TLS certificates and follow best practices for secure HTTP connections.

### Domain Control

The `did:oidf` method ties DIDs to domain names. The security of this binding depends on the security of the domain name system and the controller's ability to maintain control over their domain.

### Entity Configuration Validation

When resolving a `did:oidf` DID, implementations MUST validate the Entity Configuration according to the OpenID Federation 1.0 specification, including signature validation and expiration checking.

## Privacy Considerations

### Correlation Risk

Since `did:oidf` DIDs incorporate domain names, they inherently enable correlation across different contexts. Users of this DID method should be aware of this correlation risk.

### Identity Information Disclosure

Entity Configurations may contain identity information about the entity. Controllers should carefully consider what information they include in their Entity Configuration, as it will be publicly available through DID resolution.

### Resolution Privacy

DID resolution requires a network request to the controller's domain, which may reveal information about the resolver to the controller.

## Implementation Notes

### DID URL Parameters

The `did:oidf` method supports standard DID URL parameters as defined in the DID Core specification. Additionally, the following method-specific parameters are defined:

- `version`: Requests a specific version of the Entity Configuration
- `service`: Requests inclusion of specific service endpoints in the resolved DID Document

### Content Negotiation

Resolvers for `did:oidf` DIDs should support content negotiation to return the DID Document in different formats, including at minimum the application/did+json and application/did+ld+json media types.

## References

### Normative References

- [Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/did-core/)
- [OpenID Federation 1.0](https://openid.net/specs/openid-federation-1_0.html)
- [RFC 7517: JSON Web Key (JWK)](https://tools.ietf.org/html/rfc7517)
- [RFC 7515: JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)

### Informative References

- [Decentralized Identifier Extensions](https://www.w3.org/TR/did-extensions/)
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)

## Appendix A: Example

### Example Entity Configuration

```json
{
  "iss": "https://example.com",
  "sub": "https://example.com",
  "iat": 1615394092,
  "exp": 1646930092,
  "jwks": {
    "keys": [
      {
        "kty": "RSA",
        "use": "sig",
        "kid": "NjVBRjY5MDlCMUIwNzU4RTA2QzZFMDQ4QzQ2MDAyQjVDNjk1RTM2Qg",
        "n": "yeNlzlub94YgerT030codqEztjfU_S6X4DbDA_iVKkjAWtYfPHDzz_sPCT1Axz6isZdf3lHpq_gYX4Sz-cbe4rjmigxUxr-FgKHQy3HeCdK6hNq9ASQvMK9LBOpXDNn7mei6RZWom4wo3CMvvsY1w8tjtfLb-yQwJPltHxShZq5-ihC9irpLI9xEBTgG12q5lGIFPhTl_7inA1PFK97LuSLnTJzW0bj096v_TMDg7pOWm_zHtF53qbVsI0e3v5nmdKXdFf9BjIARRfVrbxVxiZHjU6zL6jY5QJdh1QCmENoejj_ytspMmGW7yMRxzUqgxcAqOBpVm0b-_mW3HoBdjQ",
        "e": "AQAB"
      }
    ]
  },
  "metadata": {
    "federation_entity": {
      "organization_name": "Example Organization",
      "contacts": ["operations@example.com"]
    },
    "openid_provider": {
      "issuer": "https://example.com",
      "authorization_endpoint": "https://example.com/auth",
      "token_endpoint": "https://example.com/token",
      "jwks_uri": "https://example.com/jwks"
    }
  }
}
```

### Resulting DID Document

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://identity.foundation/openid-did/v1"
  ],
  "id": "did:oidf:example.com",
  "created": "2021-03-10T14:28:12Z",
  "updated": "2022-03-10T14:28:12Z",
  "verificationMethod": [
    {
      "id": "did:oidf:example.com#NjVBRjY5MDlCMUIwNzU4RTA2QzZFMDQ4QzQ2MDAyQjVDNjk1RTM2Qg",
      "type": "JsonWebKey2020",
      "controller": "did:oidf:example.com",
      "publicKeyJwk": {
        "kty": "RSA",
        "use": "sig",
        "kid": "NjVBRjY5MDlCMUIwNzU4RTA2QzZFMDQ4QzQ2MDAyQjVDNjk1RTM2Qg",
        "n": "yeNlzlub94YgerT030codqEztjfU_S6X4DbDA_iVKkjAWtYfPHDzz_sPCT1Axz6isZdf3lHpq_gYX4Sz-cbe4rjmigxUxr-FgKHQy3HeCdK6hNq9ASQvMK9LBOpXDNn7mei6RZWom4wo3CMvvsY1w8tjtfLb-yQwJPltHxShZq5-ihC9irpLI9xEBTgG12q5lGIFPhTl_7inA1PFK97LuSLnTJzW0bj096v_TMDg7pOWm_zHtF53qbVsI0e3v5nmdKXdFf9BjIARRfVrbxVxiZHjU6zL6jY5QJdh1QCmENoejj_ytspMmGW7yMRxzUqgxcAqOBpVm0b-_mW3HoBdjQ",
        "e": "AQAB"
      }
    }
  ],
  "authentication": [
    "did:oidf:example.com#NjVBRjY5MDlCMUIwNzU4RTA2QzZFMDQ4QzQ2MDAyQjVDNjk1RTM2Qg"
  ],
  "service": [
    {
      "id": "did:oidf:example.com#openid-provider",
      "type": "OpenIdProvider",
      "serviceEndpoint": {
        "issuer": "https://example.com",
        "authorization_endpoint": "https://example.com/auth",
        "token_endpoint": "https://example.com/token",
        "jwks_uri": "https://example.com/jwks"
      }
    },
    {
      "id": "did:oidf:example.com#federation-entity",
      "type": "FederationEntity",
      "serviceEndpoint": {
        "organization_name": "Example Organization",
        "contacts": ["operations@example.com"]
      }
    }
  ]
}
```
