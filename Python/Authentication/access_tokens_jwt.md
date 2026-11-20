# Access Tokens (JWT): A Beginner's Guide

This guide explains the cheat sheet below from scratch, assuming no prior coding or Python experience. It then shows how the same ideas are built into a real project with Python and FastAPI.

![Access tokens (JWT) cheat sheet](./AuthVsAuthorization.png)

**Contents**
1. [The problem tokens solve](#1-the-problem-tokens-solve)
2. [What is an access token?](#2-what-is-an-access-token)
3. [Login to token: the 4 steps](#3-login-to-token-the-4-steps)
4. [Inside a JWT: header, payload, signature](#4-inside-a-jwt-header-payload-signature)
5. [Using the token on the next API request](#5-using-the-token-on-the-next-api-request)
6. [Authentication vs authorization](#6-authentication-vs-authorization)
7. [The "Remember" box: security rules](#7-the-remember-box-security-rules)
8. [A working project example](#8-a-working-project-example)
9. [Trying it out](#9-trying-it-out)
10. [Common mistakes and next steps](#10-common-mistakes-and-next-steps)
11. [Glossary](#11-glossary)

---

## 1. The problem tokens solve

Imagine a hotel.

- At the front desk (**login**) you show your ID and booking.
- The desk gives you a **key card** (the **token**).
- After that, you don't show your ID at every door. You tap the card, and each door checks whether the card is valid and whether it opens that room.

Websites and apps work the same way.

A web API (the server's "doors") talks to apps over **HTTP**. HTTP has no memory: each request arrives with no knowledge of earlier ones. Without tokens, the app would have to send the **username and password with every request**. That is slow and dangerous, because the password travels everywhere.

**Solution:** send the password **once**, get a temporary token back, and send the token afterwards.

---

## 2. What is an access token?

> An access token is a **temporary proof of login** sent with protected API requests.

- **Temporary:** it expires, for example after 30 minutes. If someone steals it, it only works for a short time.
- **Proof of login:** it tells the server "this request comes from someone who already logged in".
- **Protected API requests:** the token is only needed for endpoints that require login (your profile, your orders, admin pages). Public endpoints don't need one.

**JWT** stands for **JSON Web Token** (pronounced "jot"). It is the most common format for access tokens. It is a long text string like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxIiwiZXhwIjoxNzk1MTkxMjE0fQ.xGk3...
```

It looks like gibberish, but it has a clear structure (see section 4).

---

## 3. Login to token: the 4 steps

The top of the cheat sheet shows this flow between the **User** (browser or mobile app) and the **Server**:

```
User ──── username + password ────▶ Server
User ◀──────── access token ─────── Server
```

| Step | What happens | Why |
|------|--------------|-----|
| **1** | The user sends their username and password to the **login API**. | This is the only time the password is sent. |
| **2** | The server finds the user and **verifies the password hash**. | The server never stores the real password, only a scrambled version called a *hash*. |
| **3** | If correct, the server **creates and signs a token**. | Signing lets the server detect later if anyone tampered with the token. |
| **4** | The server **returns the access token**. | The app keeps it and attaches it to later requests. |

### What is a password hash?

A **hash** is a one-way scramble: easy to compute, practically impossible to reverse.

```
"wonderland"  ──hash──▶  $argon2id$v=19$m=65536,t=3,p=4$...   (can't be turned back)
```

The database stores only the hash. At login, the server hashes what the user typed and compares the two. If the database leaks, attackers get hashes, not passwords.

Use a purpose-built password hasher (Argon2 or bcrypt). Never use plain SHA or MD5 for passwords, and never store passwords as plain text.

---

## 4. Inside a JWT: header, payload, signature

A JWT is **three parts separated by dots**:

```
HEADER . PAYLOAD . SIGNATURE
```

Each of the first two parts is JSON that has been Base64URL-encoded, which is an encoding and not encryption. The third part is the signature.

### Header: "how is this token signed?"
```json
{ "alg": "HS256", "typ": "JWT" }
```
- `alg` is the **signing algorithm**. `HS256` uses one shared secret key.
- `typ` is the **token type**, which is JWT.

### Payload: "what is this token about?"
The payload holds **claims**, which are facts about the token:
```json
{ "sub": "1", "exp": 1795191214, "role": "admin" }
```
| Claim | Meaning |
|-------|---------|
| `sub` | **Subject**: who the token is about, usually the user ID. |
| `exp` | **Expiry**: when the token stops being valid, as a Unix timestamp (seconds since 1970). |
| `role` | **Optional** custom data, such as the user's role. |

### Signature: "has it been changed?"
The server computes the signature by combining the header, the payload and its **secret key**:

```
signature = HMAC_SHA256( header + "." + payload , SECRET_KEY )
```

If anyone edits the payload (for example changes `"role": "member"` to `"admin"`), the signature no longer matches. The server recomputes it, sees the mismatch and **rejects** the token. Only someone who knows the secret key can produce a valid signature.

### Signed is not the same as encrypted

This is the most important idea on the cheat sheet. A signature **prevents changes** but does **not hide** the contents. Anyone holding the token can decode and read the payload (try pasting a token into https://jwt.io). So:

- Don't put passwords or secrets in the payload.
- Treat the payload as public information.

---

## 5. Using the token on the next API request

After login, the client attaches the token to every request in the **`Authorization` header**:

```
GET /users/me HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

`Bearer` means "whoever bears (holds) this token gets access". Because holding the token is enough, keeping it safe matters.

For each request, the API does three checks (the green ticks on the cheat sheet):

1. **Checks signature and expiry.** Was the token made by us, unchanged, and not expired?
2. **Identifies the user from the claims.** `sub` says who this is.
3. **Checks permission for the requested action.** May this user do this?

If the first two fail, the API answers **401 Unauthorized**. If the third fails, it answers **403 Forbidden**.

---

## 6. Authentication vs authorization

The cheat sheet's file name is `AuthVsAuthorization`, and it labels these two words. They sound alike but ask different questions.

| | Authentication | Authorization |
|---|----------------|---------------|
| Question | **Who are you?** | **What may you do?** |
| Hotel analogy | Showing ID at the front desk | Your card opens your room but not the manager's office |
| In this guide | Login, then verify the token (steps 1 and 2) | Role or permission check (step 3) |
| Failure code | **401 Unauthorized** | **403 Forbidden** |

Example: Alice and Bob are both logged in, so both are authenticated. Only Alice is an admin, so only she is authorized to view `/admin/reports`.

---

## 7. The "Remember" box: security rules

| Rule | Why |
|------|-----|
| **JWT is signed, not encrypted: the payload can be read.** | Anyone with the token can decode it. |
| **Never put passwords or secrets in it.** | They would be exposed to anyone who sees the token. |
| **Keep the signing key secret.** | Whoever has the key can forge tokens for any user. Load it from an environment variable, never commit it to Git. |
| **Set an expiry.** | A stolen token stops working soon. |

More good habits in real projects:
- Always use **HTTPS** so tokens can't be sniffed in transit.
- Keep access tokens **short-lived** (5–30 minutes). Use a separate **refresh token** to get new ones.
- Always pass an explicit `algorithms=[...]` list when decoding, so attackers can't swap the algorithm.
- Use a long random key. Generate one with `python -c "import secrets; print(secrets.token_hex(32))"`.

---

## 8. A working project example

Below is a complete FastAPI app that implements the whole cheat sheet. It has a login endpoint that returns a JWT, a protected endpoint for any logged-in user, and an admin-only endpoint.

### Setup
```bash
python -m venv .venv
# Windows PowerShell:  .venv\Scripts\Activate.ps1
# macOS/Linux:         source .venv/bin/activate

python -m pip install "fastapi[standard]" pyjwt "pwdlib[argon2]"
```

Set the secret key (never hard-code it):
```powershell
# Windows PowerShell
$env:JWT_SECRET_KEY = python -c "import secrets; print(secrets.token_hex(32))"
```
```bash
# macOS/Linux
export JWT_SECRET_KEY=$(python -c "import secrets; print(secrets.token_hex(32))")
```

### `main.py`
```python
import os
from datetime import datetime, timedelta, timezone
from typing import Annotated

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from pwdlib import PasswordHash
from pydantic import BaseModel

# ---------- Settings ----------
SECRET_KEY = os.environ["JWT_SECRET_KEY"]   # secret: comes from the environment
ALGORITHM = "HS256"                         # the "alg" in the JWT header
ACCESS_TOKEN_EXPIRE_MINUTES = 30            # the "exp" claim: short-lived

password_hasher = PasswordHash.recommended()            # Argon2 hashing
DUMMY_HASH = password_hasher.hash("not-a-real-password")

app = FastAPI(title="JWT demo")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")  # reads "Authorization: Bearer ..."


# ---------- Data shapes ----------
class User(BaseModel):          # what we are willing to show to the outside world
    id: int
    username: str
    role: str


class Token(BaseModel):         # what the login endpoint returns
    access_token: str
    token_type: str


# ---------- A fake database (use a real one in production) ----------
FAKE_USERS = {
    1: {"id": 1, "username": "alice", "role": "admin",
        "password_hash": password_hasher.hash("wonderland")},
    2: {"id": 2, "username": "bob", "role": "member",
        "password_hash": password_hasher.hash("builder")},
}


def find_user_by_username(username: str) -> dict | None:
    for user in FAKE_USERS.values():
        if user["username"] == username:
            return user
    return None


# ---------- Step 3: create and sign the token ----------
def create_access_token(user_id: int) -> str:
    expires_at = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    payload = {"sub": str(user_id), "exp": expires_at}      # the claims
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)  # header + payload + signature


# ---------- Steps 1, 2, 4: the login endpoint ----------
@app.post("/token", response_model=Token)
def login(form: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user = find_user_by_username(form.username)              # step 2: find the user
    stored_hash = user["password_hash"] if user else DUMMY_HASH
    password_ok = password_hasher.verify(form.password, stored_hash)  # step 2: verify the hash
    if user is None or not password_ok:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",          # same message for both cases
            headers={"WWW-Authenticate": "Bearer"},
        )
    return Token(access_token=create_access_token(user["id"]), token_type="bearer")  # step 4


# ---------- Authentication: who is calling? ----------
def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]) -> User:
    unauthorized = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Invalid or expired token",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        # Checks the signature AND the expiry in one call
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = int(payload["sub"])                          # identify the user from claims
    except (InvalidTokenError, KeyError, ValueError):
        raise unauthorized
    user = FAKE_USERS.get(user_id)
    if user is None:
        raise unauthorized
    return User(**user)


CurrentUser = Annotated[User, Depends(get_current_user)]


# ---------- Authorization: what may they do? ----------
def require_admin(user: CurrentUser) -> User:
    if user.role != "admin":
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Admins only")
    return user


# ---------- Protected endpoints ----------
@app.get("/users/me", response_model=User)
def read_me(user: CurrentUser):                  # any logged-in user
    return user


@app.get("/admin/reports")
def read_reports(admin: Annotated[User, Depends(require_admin)]):   # admins only
    return {"report": "secret numbers", "requested_by": admin.username}
```

### How the code maps to the cheat sheet

| Cheat sheet | Code |
|-------------|------|
| 1. User sends username and password | `login(form: ...)` receives the form fields |
| 2. Server finds user, verifies password hash | `find_user_by_username` and `password_hasher.verify` |
| 3. Server creates and signs a token | `create_access_token` calls `jwt.encode` |
| 4. Server returns the access token | `return Token(...)` |
| Header | Added automatically by `jwt.encode` (`alg`, `typ`) |
| Payload (`sub`, `exp`) | the `payload` dictionary |
| Signature | produced by `jwt.encode` using `SECRET_KEY` |
| `Authorization: Bearer <token>` | `OAuth2PasswordBearer` extracts the token |
| API checks signature and expiry | `jwt.decode(...)` raises `InvalidTokenError` on failure |
| Identifies user from claims | `payload["sub"]` then the user lookup |
| Checks permission | `require_admin` |
| Authentication | `get_current_user` (returns 401) |
| Authorization | `require_admin` (returns 403) |

### Python concepts used (quick explanations)

- **`import`**: loads code written by others (libraries).
- **`def name(...):`**: defines a **function**, a reusable block of instructions.
- **`class User(BaseModel)`**: defines a data shape. FastAPI uses it to validate data automatically.
- **`@app.post("/token")`**: a **decorator** that registers the function below it as a web endpoint.
- **`Depends(...)`**: **dependency injection**. FastAPI runs that function first and passes its result in. It is how `get_current_user` runs before every protected endpoint.
- **`try / except`**: tries something and handles the failure. Here a bad token becomes a clean 401 instead of a crash.
- **`raise HTTPException(...)`**: stops the request and returns an HTTP error to the caller.
- **Type hints (`user_id: int`)**: label what kind of value is expected. FastAPI uses them for validation.

### Design notes for real projects
- **Same error for "wrong user" and "wrong password".** This stops attackers from discovering which usernames exist. The `DUMMY_HASH` makes both cases take similar time.
- **The role is read from the database, not from the token.** A token is valid until it expires, so reading the role fresh means a demoted admin loses access immediately. Putting `role` in the token is allowed (the cheat sheet marks it optional) but then it can be stale.
- **The fake `FAKE_USERS` dictionary** stands in for a real database such as PostgreSQL. Only that lookup code changes in a real project.

---

## 9. Trying it out

Start the server (with `JWT_SECRET_KEY` set):
```bash
fastapi dev main.py
```
Then open **http://127.0.0.1:8000/docs**. Click **Authorize**, log in as `alice` / `wonderland`, and try the endpoints in the browser.

Or use the command line (`curl.exe` on Windows PowerShell):

**1. Log in and get a token**
```bash
curl -X POST http://127.0.0.1:8000/token -d "username=alice&password=wonderland"
```
```json
{"access_token":"eyJhbGciOi...","token_type":"bearer"}
```

**2. Call a protected endpoint with the token**
```bash
curl http://127.0.0.1:8000/users/me -H "Authorization: Bearer <paste token here>"
```
```json
{"id":1,"username":"alice","role":"admin"}
```

**What you should see**

| Request | Result |
|---------|--------|
| Login with right password | `200` and a token |
| Login with wrong password or unknown user | `401` |
| `/users/me` with no token | `401` |
| `/users/me` with a tampered token | `401` (signature mismatch) |
| `/users/me` with an expired token | `401` |
| `/admin/reports` as **bob** (member) | `403` (authenticated but not authorized) |
| `/admin/reports` as **alice** (admin) | `200` |

I ran each of these scenarios against this exact code with FastAPI's test client, and they behave as listed.

### Peek inside your own token
```python
import base64, json

token = "<paste token>"
header_b64, payload_b64, signature = token.split(".")

def decode(part):
    return json.loads(base64.urlsafe_b64decode(part + "=" * (-len(part) % 4)))

print(decode(header_b64))    # {'alg': 'HS256', 'typ': 'JWT'}
print(decode(payload_b64))   # {'sub': '1', 'exp': 1795191214}
```
This shows that the payload is readable by anyone, which is why it must never contain secrets.

---

## 10. Common mistakes and next steps

**Common mistakes**
- Hard-coding `SECRET_KEY` in the source code, or committing it to Git.
- Storing plain-text passwords, or hashing them with a fast hash like MD5.
- Putting sensitive data in the payload.
- Tokens that never expire.
- Forgetting `algorithms=[...]` when decoding.
- Returning 401 when the user is logged in but lacks permission. That should be 403.
- Sending tokens over plain HTTP.

**Where to go next**
- **Refresh tokens:** long-lived tokens used only to get new short-lived access tokens.
- **Token storage on the client:** in a browser, an `HttpOnly` cookie is generally safer than `localStorage`, which JavaScript can read.
- **Logout and revocation:** JWTs can't be "un-issued", so use short expiry, or keep a deny list of revoked tokens.
- **Asymmetric signing (`RS256`):** a private key signs and a public key verifies, which is useful when many services need to verify tokens without being able to create them.
- **OAuth2 / OpenID Connect:** "Log in with Google/GitHub" builds on the same token ideas.
- Official guide: [FastAPI security tutorial](https://fastapi.tiangolo.com/tutorial/security/).

---

## 11. Glossary

| Term | Plain-English meaning |
|------|----------------------|
| **API** | A set of "doors" (URLs) that programs use to talk to a server. |
| **HTTP** | The language browsers and apps use to talk to servers. |
| **Endpoint** | One specific API door, such as `/users/me`. |
| **Authentication** | Proving who you are. |
| **Authorization** | Deciding what you are allowed to do. |
| **Token** | A temporary digital key card. |
| **JWT** | A token format made of header, payload and signature. |
| **Claim** | One fact inside the payload (`sub`, `exp`, `role`). |
| **`sub`** | Subject: the user the token is about. |
| **`exp`** | Expiry: when the token stops working. |
| **Hash** | A one-way scramble of data, used to store passwords safely. |
| **Signature** | A tamper-detection stamp made with the secret key. |
| **Secret key** | The private value the server uses to sign tokens. |
| **Bearer** | "Whoever holds this token is let in." |
| **401 Unauthorized** | "I don't know who you are" (missing or bad token). |
| **403 Forbidden** | "I know who you are, but you may not do this." |
| **Environment variable** | A setting stored outside the code, good for secrets. |
