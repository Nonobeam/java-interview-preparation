# 48. Symmetric vs Asymmetric Encryption

The core difference: **one key vs two keys**.

---

## Symmetric encryption

Same key is used to both encrypt and decrypt.

```
Sender:   plaintext + key  →  ciphertext
Receiver: ciphertext + key →  plaintext
```

- Algorithms: **AES** (AES-128, AES-256), DES, 3DES
- Fast — suited for bulk data encryption
- Problem: **key distribution** — how do you securely share the key?

```java
// AES example
SecretKey key = KeyGenerator.getInstance("AES").generateKey();  // 128-bit

Cipher cipher = Cipher.getInstance("AES");
cipher.init(Cipher.ENCRYPT_MODE, key);
byte[] encrypted = cipher.doFinal("hello".getBytes());

cipher.init(Cipher.DECRYPT_MODE, key);
byte[] decrypted = cipher.doFinal(encrypted);
```

---

## Asymmetric encryption

Two mathematically linked keys: **public key** (encrypt / verify) and **private key** (decrypt / sign).

```
Encryption:  plaintext  + publicKey  → ciphertext
Decryption:  ciphertext + privateKey → plaintext

Signing:     data       + privateKey → signature
Verification:signature  + publicKey  → valid/invalid
```

- Algorithms: **RSA**, EC (Elliptic Curve), DSA
- Slower — used for small payloads or key exchange
- Solves key distribution: share the public key freely, keep private key secret

```java
// RSA key pair generation
KeyPairGenerator gen = KeyPairGenerator.getInstance("RSA");
gen.initialize(2048);
KeyPair pair = gen.generateKeyPair();
PublicKey  pub  = pair.getPublic();
PrivateKey priv = pair.getPrivate();

// Sign
Signature sig = Signature.getInstance("SHA256withRSA");
sig.initSign(priv);
sig.update(data);
byte[] signature = sig.sign();

// Verify
sig.initVerify(pub);
sig.update(data);
boolean valid = sig.verify(signature);
```

---

## How they combine in TLS (HTTPS)

```
1. Asymmetric (RSA/EC) — used during handshake to exchange a session key
2. Symmetric (AES)     — used for the actual data transfer (fast)
```

TLS never sends bulk data with RSA — it's too slow. It only uses RSA to establish the shared AES key.

---

## JWT connection

| JWT Algorithm | Type | Key used |
|---|---|---|
| HS256 / HS512 | Symmetric (HMAC) | One shared secret |
| RS256 / RS512 | Asymmetric (RSA) | Private key signs, public key verifies |
| ES256 | Asymmetric (EC) | Private key signs, public key verifies |

**When to choose RS256 over HS256:**
- Multiple services need to **verify** tokens without being able to **issue** them.
- Share the public key with all services; keep the private key only on the auth server.
- HS256 requires every verifier to know the secret — if one service is compromised, all tokens are at risk.

```java
// RS256 JWT signing
PrivateKey privateKey = loadPrivateKey("private.pem");
String token = Jwts.builder()
    .subject("user123")
    .signWith(privateKey, Jwts.SIG.RS256)
    .compact();

// Verifying (any service, only needs public key)
PublicKey publicKey = loadPublicKey("public.pem");
Claims claims = Jwts.parser()
    .verifyWith(publicKey)
    .build()
    .parseSignedClaims(token)
    .getPayload();
```

---

## Quick comparison

| | Symmetric | Asymmetric |
|---|---|---|
| Keys | 1 shared key | Public + Private |
| Speed | Fast | Slow |
| Use case | Bulk data (AES) | Key exchange, signatures |
| Key distribution | Hard | Easy (public key is public) |
| JWT algorithm | HS256 | RS256, ES256 |

---

## Interview one-liner
> "Symmetric uses one shared key — fast but hard to distribute securely. Asymmetric uses a public/private key pair — the private key signs or decrypts, the public key verifies or encrypts. TLS combines both: asymmetric for key exchange, then symmetric for bulk transfer. JWT uses HS256 (symmetric) for single-service auth or RS256 (asymmetric) when multiple services need to verify tokens without being able to issue them."
