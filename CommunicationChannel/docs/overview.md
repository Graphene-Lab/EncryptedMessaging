# CommunicationChannel – System Overview

## Purpose

**CommunicationChannel** is a low-level, transport-agnostic communication library. It establishes reliable, full-duplex byte-stream channels between peers and provides the networking foundation for **EncryptedMessaging** and the entire cloud stack.

By design, the library is **medium-independent**: the default implementation uses TCP sockets, but the abstraction allows replacing the transport with GSM data networks, RS-232/RS-485 serial links, LoRa, acoustic modems, or any other byte-oriented medium without changing any higher-level code.

## Architecture

```
EncryptedMessaging / CloudSync
		│
		▼
Channel  (high-level API: connect, send, receive events)
		│
		├── DataIO / KeepAlive / TimerAutoDisconnect / TimerTryReconnection
		│
		▼
IDataConnection  (transport abstraction)
		├── TcpClientConnection   (default – TCP socket)
		└── NamedPipeClientConnection  (IPC / local)
```

## Key Features

- **Automatic reconnection** – `TimerTryReconnection` retries lost connections transparently.
- **Keep-alive** – `KeepAlive` sends heartbeat packets to detect silent disconnections.
- **Auto-disconnect** – `TimerAutoDisconnect` drops idle connections after a configurable timeout.
- **Anti-duplicate** – `AntiDuplicate` discards retransmitted packets (idempotent delivery).
- **Protocol framing** – `Protocol.cs` handles length-prefixed packet framing over the byte stream.
- **Multi-domain** – `Channel` supports independent domains sharing the same physical connection.

## Transport Abstraction

Implement `IDataConnection` to add a new transport medium:

```csharp
public interface IDataConnection
{
	void Connect(string host, int port);
	void Disconnect();
	void Send(byte[] data);
	event Action<byte[]> DataReceived;
	bool IsConnected { get; }
}
```

Register your implementation when creating a `Channel` instance.

## Security Note

CommunicationChannel intentionally provides **no encryption**. Encryption is the responsibility of the upper layer (EncryptedMessaging). This separation of concerns means the channel can be audited and replaced independently.

## Target Framework

**.NET Standard 2.1**
