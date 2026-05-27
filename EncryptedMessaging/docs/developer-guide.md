# EncryptedMessaging – Developer Guide

## Installation

```bash
dotnet add package EncryptedMessaging
```

## Quick Start

```csharp
using EncryptedMessaging;

// 1. Create a context (generates a new identity if no passphrase is stored)
var context = new Context(
	entryPoint: "server.example.com:port",
	networkName: "MyApp",
	passphrase: null,           // null = generate new identity
	onMessageArrived: OnMessage
);

// 2. Add a contact by their public key
var contact = context.Contacts.AddContact(publicKeyBytes, "Alice");

// 3. Send a message
context.Messaging.SendText(contact, "Hello, Alice!");

// 4. Receive messages
void OnMessage(Message message)
{
	Console.WriteLine($"From {message.Contact.Name}: {message.GetText()}");
}
```

## Identity Management

The identity (key-pair) is generated once from a random or user-supplied BIP39 passphrase:

```csharp
// Retrieve the current passphrase (save this securely!)
string passphrase = context.My.Passphrase;

// Restore an existing identity
var context = new Context(..., passphrase: "word1 word2 ... word12");
```

## Groups

```csharp
// Create a group
var group = context.Contacts.CreateGroup("Team", new[] { alice, bob });

// Send to group (single encrypted payload, per-member key wrapping)
context.Messaging.SendText(group, "Hello team!");
```

## Custom Transport (DataChannel)

```csharp
public class MyChannel : ChannelBaseClass
{
	protected override void Send(byte[] data) { /* transmit bytes */ }
	// override Receive event to inject incoming bytes
}

var context = new Context(..., dataChannel: new MyChannel());
```

## Message Types

Use `MessageFormat` constants or the typed helpers:

```csharp
context.Messaging.SendText(contact, "text");
context.Messaging.SendData(contact, bytes, MessageFormat.Binary);
```

## OEM Customization

`OEM.cs` provides hooks for white-label deployments: custom app name, server address, and branding.

## Settings

`Setting.cs` exposes persistent configuration (stored via `SecureStorage`):

```csharp
context.Setting.Notifications = true;
```

## Building

```powershell
dotnet build ..\..\EncryptedMessaging\EncryptedMessaging\EncryptedMessaging.csproj
```
