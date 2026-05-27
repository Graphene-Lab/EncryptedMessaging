# EncryptedMessaging – System Overview

## Purpose

**EncryptedMessaging** is a secure, trustless communication library for .NET. It provides end-to-end AES-256 encrypted messaging with per-message ephemeral keys, digital identity management based on Bitcoin cryptography, and group multicast support — all without requiring a central account server.

The library has been proposed as a **STANG-V2 (Secure Tactical Adaptive Next-Generation Protocol)** candidate, suitable for military, industrial, and high-security civilian applications.

## Core Concepts

### Digital Identity
Each participant is identified by an **ECDSA key-pair** derived from a BIP39 passphrase (same technology as Bitcoin wallets). The public key is the identity; the private key never leaves the device.

### Per-Message Ephemeral Keys (EPMKE)
Every message is encrypted with a unique ephemeral AES-256-GCM key. The key is then individually wrapped (asymmetrically encrypted) for each recipient's public key. Compromise of one message key does not affect any other message.

### Group Communications (GAKD)
Groups are identified by a hash of their members' public keys (irreversible). A single encrypted payload with per-member key wrappers enables efficient multicast without key reuse.

### Transport Agnosticism
The library abstracts the transport layer through `DataChannel` implementations:
- `MemoryDataChannel` – in-process (testing / IPC)
- `PushPullDataChannel` – polling-based (HTTP, file, etc.)
- Custom channels can be added by implementing `ChannelBaseClass`

The underlying `CommunicationChannel` library provides the default TCP socket transport, but any medium (GSM, LoRa, RS-232, …) can be substituted.

## Architecture

```
Application
	│
	▼
Context  ──────────────► Contacts (Contact list)
	│                        │
	▼                        ▼
Messaging                 Contact (public key + metadata)
	│
	▼
CryptoServiceProvider  (AES-256-GCM, ECDSA-521)
	│
	▼
DataChannel  ──► CommunicationChannel (TCP) or custom transport
```

## Key Files

| File | Role |
|---|---|
| `Context.cs` | Library entry point; holds identity, contacts, messaging |
| `Contact.cs` | Represents a communication peer |
| `Contacts.cs` | Contact list management |
| `Messaging.cs` | Message send/receive orchestration |
| `CryptoServiceProvider.cs` | Cryptographic primitives |
| `MessageFormat.cs` | Binary message serialization |
| `Repository.cs` | Persistent message storage |
| `Time.cs` | Tamper-resistant timestamp |
| `CloudManager.cs` | Integration with cloud-based message routing |

## Security Highlights

- AES-256-GCM bulk encryption.
- ECDSA-521 digital signatures on every packet.
- Key derivation ratcheting (each key derived from the previous, non-reversible).
- No server-side contact list; server is a stateless router.
- Group IDs are hashed from member public keys — group membership cannot be reverse-engineered from the ID.
- Anonymised participant identifiers.

## Localisation

Resource files provide translations for DE, ES, FR, IT, and EN (default).

## Target Framework

**.NET Standard 2.1**
