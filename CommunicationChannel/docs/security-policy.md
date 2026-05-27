# CommunicationChannel – Security Policy

## Security Model

CommunicationChannel is intentionally **transport-only** — it provides no encryption. All security guarantees are delegated upward to the **EncryptedMessaging** layer, which encrypts and digitally signs every packet before it reaches CommunicationChannel.

This separation of concerns means:

- The channel implementation can be audited independently of the cryptographic layer.
- The channel can be replaced (e.g., TCP → GSM → LoRa) without affecting the security model.
- A compromised channel cannot expose plaintext: the data it carries is already AES-256 encrypted.

## Attack Surface

| Component | Risk | Mitigation |
|---|---|---|
| TCP socket | Traffic interception | Upper-layer AES-256-GCM encryption |
| TCP socket | Packet injection | ECDSA-521 signatures in EncryptedMessaging |
| Auto-reconnect | Reconnection to rogue server | Server authenticated via public key (QR pairing) |
| AntiDuplicate window | Memory exhaustion | Sliding window with configurable max size |
| Named pipe (IPC) | Local process injection | OS ACLs on the pipe; restrict to trusted processes |

## Reporting a Vulnerability

1. **Do not** open a public GitHub issue.
2. Contact the maintainer privately (see repository contacts).
3. Allow up to **90 days** for a fix before public disclosure.

## Supported Versions

Only the latest released version receives security patches.
