# EncryptedMessaging – Testing Strategy

## Overview

EncryptedMessaging is a security-critical library. The testing strategy prioritises:

1. **Cryptographic correctness** – encrypted data can be decrypted only by the intended recipient.
2. **Protocol correctness** – messages are delivered reliably, in order, without duplicates.
3. **Group multicast** – all group members receive the message; non-members cannot decrypt it.
4. **Transport independence** – switching from TCP to `MemoryDataChannel` does not affect correctness.

---

## 1. Cryptographic Unit Tests

```csharp
[Fact]
public void EncryptDecrypt_RoundTrip_Succeeds()
{
	var alice = new CryptoServiceProvider();
	var bob   = new CryptoServiceProvider();

	byte[] plaintext  = Encoding.UTF8.GetBytes("Secret message");
	byte[] ciphertext = alice.Encrypt(plaintext, bob.PublicKey);
	byte[] decrypted  = bob.Decrypt(ciphertext);

	Assert.Equal(plaintext, decrypted);
}

[Fact]
public void Signature_Verification_Succeeds()
{
	var alice = new CryptoServiceProvider();
	byte[] data = Encoding.UTF8.GetBytes("Document");
	byte[] sig  = alice.Sign(data);
	Assert.True(alice.Verify(data, sig, alice.PublicKey));
}

[Fact]
public void Signature_Tampered_Data_Fails()
{
	var alice = new CryptoServiceProvider();
	byte[] data    = Encoding.UTF8.GetBytes("Document");
	byte[] sig     = alice.Sign(data);
	byte[] tampered = Encoding.UTF8.GetBytes("Modified");
	Assert.False(alice.Verify(tampered, sig, alice.PublicKey));
}
```

---

## 2. Messaging Integration Tests (MemoryDataChannel)

Use `MemoryDataChannel` for in-process, zero-network tests:

```csharp
[Fact]
public async Task SendText_Alice_To_Bob_Delivered()
{
	var (alice, bob) = CreatePair(); // two contexts sharing MemoryDataChannel
	string received = null;
	bob.OnMessageArrived = msg => received = msg.GetText();

	alice.Messaging.SendText(alice.Contacts.Get("bob"), "Hello Bob");

	await Task.Delay(100); // allow async dispatch
	Assert.Equal("Hello Bob", received);
}
```

---

## 3. Group Multicast Tests

```csharp
[Fact]
public async Task GroupMessage_DeliveredToAllMembers_NotToOutsider()
{
	var (alice, bob, carol, eve) = CreateFourContexts();
	var group = alice.Contacts.CreateGroup("team", new[] { bob.Contact, carol.Contact });

	var received = new List<string>();
	bob.OnMessageArrived   = msg => received.Add("bob");
	carol.OnMessageArrived = msg => received.Add("carol");
	eve.OnMessageArrived   = msg => received.Add("eve");

	alice.Messaging.SendText(group, "Team message");
	await Task.Delay(100);

	Assert.Contains("bob",   received);
	Assert.Contains("carol", received);
	Assert.DoesNotContain("eve", received);
}
```

---

## 4. Replay Attack Test

```csharp
[Fact]
public void DuplicateMessage_IsDiscarded()
{
	// Inject the same serialized packet twice; assert it is received only once
}
```

---

## Test Coverage Goals

| Area | Target |
|---|---|
| Cryptographic primitives | 100 % |
| Message send/receive | ≥ 90 % |
| Group multicast | ≥ 90 % |
| Digital signatures | 100 % |
| Transport switching | ≥ 80 % |

---

## Running Tests

```powershell
dotnet test ..\..\EncryptedMessaging\EncryptedMessaging\
```
