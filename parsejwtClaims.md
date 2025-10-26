Ah! Let’s break down **how `parseClaimsJws` actually parses the claims** in your JWT code, step by step.

---

## **1. JWT Structure Reminder**

A JWT looks like this:

```
header.payload.signature
```

* **Header** → JSON describing the algorithm and type:

  ```json
  { "alg": "HS256", "typ": "JWT" }
  ```
* **Payload (Claims)** → JSON containing the information you put in the token:

  ```json
  { "sub": "user123", "type": "ACCESS", "iat": 1690000000, "exp": 1690009000 }
  ```
* **Signature** → HMAC SHA-256 hash of `(base64Url(header) + "." + base64Url(payload))` using your secret key.

---

## **2. What happens in `parseClaimsJws`**

```kotlin
Jwts.parserBuilder()
    .setSigningKey(secretKey)
    .build()
    .parseClaimsJws(token)
    .body
```

1. **`parserBuilder()`** → Creates a JWT parser object.

2. **`setSigningKey(secretKey)`** → Tells the parser which key to use to **verify the token’s signature**.

3. **`build()`** → Builds the parser.

4. **`parseClaimsJws(token)`** → Core step:

   * Splits JWT into header, payload, signature.
   * Decodes header & payload from Base64URL.
   * Recomputes the HMAC SHA-256 signature using your secret key.
   * Compares computed signature with the signature in the token.
   * If signature is valid and token is not expired, it **parses the payload into a `Claims` object**.
   * If invalid → throws `JwtException`.

5. **`.body`** → Returns the **Claims object** (payload) for your token.

   * You can now do `claims.subject` → `sub`
   * Or `claims["type"]` → custom claim

---

### **3. Example**

If you generate a token:

```json
{
  "sub": "user123",
  "type": "ACCESS",
  "iat": 1690000000,
  "exp": 1690009000
}
```

Then `parseClaimsJws(token).body` will give you a `Claims` object where:

```kotlin
val claims = parseAllClaims(token)
println(claims.subject)  // "user123"
println(claims["type"])  // "ACCESS"
println(claims.expiration) // Date object of expiry
```

---

### **4. Security Checks during Parsing**

* **Signature check** → ensures token wasn’t tampered.
* **Expiration check** → ensures token is still valid (`exp` claim).
* Optional: you can also check **not before (`nbf`)** claim if you add it.

---

### ✅ **Takeaways**

1. `parseClaimsJws` **both validates and parses** the token.
2. The returned `Claims` object is just a **map-like object** of all payload fields.
3. If the token is **invalid, expired, or tampered**, it throws an exception.

---

If you want, I can **draw a small diagram showing JWT parsing flow step by step**—header, payload, signature, and how `parseClaimsJws` verifies it.

Do you want me to do that?
