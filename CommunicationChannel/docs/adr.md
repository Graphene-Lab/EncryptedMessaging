# CommunicationChannel – Architecture Decision Records

## ADR-001 – Transport Abstraction via IDataConnection Interface

**Status:** Accepted

**Context:** The messaging system needs to support multiple transport types (TCP, Named Pipes) without the higher-level protocol layer caring about the underlying transport.

**Decision:** Define `IDataConnection` as the transport abstraction. Concrete implementations (`TcpClientConnection`, `NamedPipeClientConnection`) handle the transport specifics. The `Channel` class operates exclusively on `IDataConnection`.

**Consequences:** Adding a new transport (e.g., WebSocket) requires only a new `IDataConnection` implementation. Existing protocol code is untouched.

---

## ADR-002 – Anti-Duplicate Message Filter

**Status:** Accepted

**Context:** In unreliable network conditions, messages may be delivered more than once. Higher-level handlers should not process duplicate messages.

**Decision:** `AntiDuplicate.cs` maintains a sliding window of recently seen message IDs. Duplicate messages are silently dropped at the channel layer.

**Consequences:** Deduplication is transparent to all consumers. The sliding window has a fixed maximum size to bound memory usage.

---

## ADR-003 – Keep-Alive with Auto-Reconnect

**Status:** Accepted

**Context:** TCP connections can silently drop (NAT timeout, network change, server restart). Clients must detect and recover from disconnections without user intervention.

**Decision:** `DataIO/KeepAlive.cs` sends periodic heartbeat packets. `DataIO/TimerTryReconnection.cs` automatically attempts reconnection when a keep-alive fails or a disconnect is detected.

**Consequences:** Connections are self-healing. The reconnect timer introduces brief gaps in message delivery during reconnection. `TimerAutoDisconnect.cs` handles the case where the session should expire rather than reconnect (mobile/battery saving mode).

---

## ADR-004 – Frame-Based Binary Protocol

**Status:** Accepted

**Context:** TCP is a stream protocol; messages must be framed. A text-based protocol (JSON, XML) would add encoding overhead for binary payloads (file chunks, encrypted data).

**Decision:** `Protocol.cs` defines a compact binary framing format with a fixed-size header containing the message length and type flags. Payloads are raw bytes.

**Consequences:** Efficient for binary data. Protocol changes require versioning. The 16 MB maximum message size (enforced at `RouterServer`) is a hard limit imposed by this frame format.

---

## ADR-005 – Commands for Server Kept Separate

**Status:** Accepted

**Context:** Some commands are directed at the router/server infrastructure itself rather than being forwarded to a destination client. Mixing them with forwarded messages would complicate dispatch logic.

**Decision:** `CommandsForServer.cs` defines the set of commands handled locally by the server. These are distinguished by a flag in the message header.

**Consequences:** Clean separation between infrastructure commands and application-level messages. Enables the router to act on infrastructure commands (e.g., login, ping) without forwarding them.
