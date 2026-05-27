# EncryptedMessaging – Security Policy

## Security Model

EncryptedMessaging is designed for hostile environments (military, industrial, high-security civilian). The security model provides:

- **AES-256-GCM** per-message encryption with ephemeral keys.
- **ECDSA-521** digital signatures on every packet (prevents man-in-the-middle).
- **Key derivation ratcheting**: each message key is derived from the previous, non-reversible.
- **Zero server-side account**: identity = ECDSA key-pair; no password database.
- **Anonymous group IDs**: group ID = hash of member public keys (irreversible).
- **No plaintext on router**: the routing server is stateless and cannot decrypt messages.

## Threat Model

| Threat | Mitigation |
|---|---|
| Passive eavesdropping | AES-256-GCM encryption |
| Man-in-the-middle | ECDSA-521 packet signatures |
| Server breach | Server holds only ciphertext |
| Single message key compromise | Ephemeral per-message keys |
| Future quantum attacks | Hybrid ECDSA-521 + NTRU (roadmap) |
| Metadata analysis | Pseudonymous IDs derived from public key hashes |
| Replay attacks | Per-message ephemeral keys + timestamp validation |

## Reporting a Vulnerability

1. **Do not** open a public GitHub issue for security vulnerabilities.
2. Contact the maintainer privately (see repository contacts).
3. Include: affected version, description, reproduction steps, impact assessment.
4. Allow up to **90 days** for remediation before public disclosure.

## Supported Versions

Only the latest released version receives security patches. Update promptly.

## Dependency Audit

Run periodically to check for known CVEs in NuGet dependencies:

```bash
dotnet list package --vulnerable
```
