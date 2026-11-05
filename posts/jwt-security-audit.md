# JWT security: how to audit a token by hand

A JSON Web Token is three base64url chunks separated by dots:
header, payload, signature. Anyone can decode the first two. That
single fact explains half of all JWT findings, so every audit starts
there.

## Step 1: decode it and read the claims

Paste the token into any decoder, or use a local tool so tokens from
real assessments never leave your machine. Read:

- `alg` in the header: what algorithm the server claims to accept
- `exp`: when the token expires, and whether it exists at all
- `iat`: issued at. A token issued in the future means validation is
  sloppy somewhere
- `iss` and `aud`: who issued it, and who it is meant for
- The payload itself: I regularly find roles, tenant IDs, internal
  URLs, and once in a while something that should never have left the
  database

A token is a bearer credential printed in a readable envelope.
Anything sensitive in the payload is public to anyone who holds it.

## Step 2: the classic attacks, checked in order

**alg: none.** Change the header to `"alg":"none"`, strip the
signature, keep the trailing dot. If the server accepts it, signature
verification is effectively off. Still found, still critical.

**Weak HMAC secret.** HS256 tokens are signed with a shared secret.
If that secret is `secret`, `changeme`, or the company name, anyone
can forge any user, including admins. Test the notorious defaults
offline against the captured token; a match is a one-line proof.

**Algorithm confusion.** If the header can be switched from RS256 to
HS256, some libraries will verify the "HMAC" using the public RSA
key, which is public. That converts an asymmetric design into a
symmetric one with a known secret.

**Expiry and lifetime.** No `exp`, or an `exp` ten years out, means a
leaked token is a permanent credential. Tokens should live minutes to
hours, with refresh handled separately.

**Kid and path tricks.** A `kid` header that is treated as a file
path or SQL input is an injection wearing a JWT costume. Worth two
minutes on any assessment.

## Step 3: what I run

My analyzer decodes the token locally, flags every issue above that
can be decided from the token alone, and tests the common weak
secrets for HS256:

```bash
git clone https://github.com/RahilSahu/jwt-analyzer
python3 jwt_analyze.py "eyJhbGciOi..."
```

It says this in its own output, and it is the most important line in
JWT security: **decoding is not verification**. A token that decodes
cleanly has told you its claims, nothing about whether to trust them.
Trust comes only from verifying the signature with the right key and
validating the claims, on the server, on every request.

## Fix list

- Verify the signature with an algorithm allowlist, never with the
  algorithm the token asks for
- Reject `none`, always
- Use a long random secret for HMAC, or asymmetric keys managed properly
- Short expiry, validate `exp`, `iss` and `aud` on every request
- Keep sensitive data out of the payload entirely
