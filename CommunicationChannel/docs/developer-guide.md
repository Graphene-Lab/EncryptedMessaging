# CommunicationChannel – Developer Guide

## Basic Usage

```csharp
using CommunicationChannel;

// Create a channel connected to a router/server
var channel = new Channel(
	entryPoint: "router.example.com",
	port: 5050,
	domain: 0,          // logical domain ID
	onDataReceived: bytes => { /* handle incoming data */ },
	id: myPublicKeyHash
);

// Send data
channel.SendData(payloadBytes);

// Disconnect
channel.Dispose();
```

## Constructor Parameters

| Parameter | Type | Description |
|---|---|---|
| `entryPoint` | `string` | Host name or IP of the router |
| `port` | `int` | TCP port |
| `domain` | `ulong` | Logical domain (allows multiple independent circuits over one connection) |
| `onDataReceived` | `Action<byte[]>` | Callback for incoming data |
| `id` | `ulong` | Identifier derived from the public key hash |

## Reconnection Behaviour

- `TimerTryReconnection` automatically retries on disconnect.
- The retry interval is configurable via `Channel.ReconnectionIntervalMs`.
- `OnConnectivityChange` event fires on connect/disconnect transitions.

## Using Named Pipes (IPC)

```csharp
var connection = new NamedPipeClientConnection("MyPipeName");
var channel = new Channel(connection, domain: 0, onDataReceived: ...);
```

## Implementing a Custom Transport

1. Implement `IDataConnection`.
2. Pass the instance to the `Channel` constructor overload that accepts `IDataConnection`.

## Anti-Duplicate

`AntiDuplicate` maintains a sliding window of recently seen packet hashes. Duplicate packets (e.g., from retransmissions) are silently dropped. The window size is configurable.

## Commands for Server

`CommandsForServer.cs` contains the binary command set used between client and the routing server. These are low-level protocol commands (ping, subscribe, etc.) and should not be called directly by application code.

## Building

```powershell
dotnet build ..\..\EncryptedMessaging\CommunicationChannel\CommunicationChannel.csproj
```
