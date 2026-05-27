# EncryptedMessaging – Architecture Decision Records

## ADR-001: Per-Message Ephemeral Key Encryption (EPMKE)

**Date**: 2024  
**Status**: Accepted

### Context
Session-based encryption (e.g., TLS) derives a session key that is reused for all messages in a session. Compromise of the session key exposes all messages in that session.

### Decision
Generate a **unique AES-256-GCM key for every message**. Each key is wrapped (asymmetrically encrypted) with the recipient's public key and included in the packet header. The message key is never stored.

### Consequences
- **Positive**: compromise of one message key exposes only that message.
- **Positive**: no session state to steal.
- **Negative**: each packet is larger (key wrapping adds overhead per recipient).

---

## ADR-002: Group-Aware Key Distribution (GAKD)

**Date**: 2024  
**Status**: Accepted

### Context
Multicast to N recipients would naively require N separate encrypted transmissions.

### Decision
A single AES-encrypted payload is combined with N asymmetrically-wrapped copies of the AES key (one per recipient). The router delivers one packet; each recipient decrypts their copy of the key and then decrypts the payload.

### Consequences
- **Positive**: bandwidth is O(payload + N × key_wrap) instead of O(N × payload).
- **Positive**: the payload itself is encrypted once, reducing CPU load on the sender.
- **Negative**: packet size grows linearly with group size (acceptable for tactical group sizes).

---

## ADR-003: No Server-Side Contact Lists

**Date**: 2024  
**Status**: Accepted

### Context
A server-side contact list is a high-value target: it reveals the social graph of all users even if message content is encrypted.

### Decision
Contact lists are stored **exclusively on the client**, encrypted by `SecureStorage`. The server (router) sees only sender/recipient IDs derived from public key hashes — it cannot reconstruct the social graph.

### Consequences
- **Positive**: server breach does not reveal relationships between users.
- **Positive**: no GDPR-sensitive personal data on the server.
- **Negative**: contact list is lost if the device is lost and no backup of the encrypted store exists.

---

## ADR-004: Transport Layer Agnosticism

**Date**: 2024  
**Status**: Accepted

### Context
Military and industrial scenarios may not have internet access. Communications may need to occur over GSM, LoRa, RS-232, or acoustic links.

### Decision
Abstract the transport behind a `DataChannel` interface. The default implementation uses TCP via `CommunicationChannel`, but any `ChannelBaseClass` implementation can be substituted.

### Consequences
- **Positive**: the cryptographic layer is completely independent of the physical medium.
- **Positive**: new hardware support can be added without touching EncryptedMessaging.
- **Negative**: higher-bandwidth features (e.g., large file transfer) must be rate-limited on low-bandwidth channels.
