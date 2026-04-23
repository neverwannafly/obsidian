## What OAuth 2.0 Solves  
  
OAuth 2.0 allows an application to get **limited access** to a user’s account or protected resources **without asking for their password**.  
  
Instead of:  
  
> “Give me your password”  
  
it becomes:  
  
> “Log in on the provider’s system and approve access, then I’ll receive a temporary token.”  
  
### Example Use Cases  
  
- Accessing GitHub repositories  
- Accessing Google Drive files  
- Connecting an IDE like Cursor or Claude Desktop to an MCP server  
- Third-party integrations with APIs  
  
---  
  
## Flow Used Here: Authorization Code Flow + PKCE  
  
This is the modern secure OAuth flow used for:  
  
- Browser apps  
- Mobile apps  
- CLI tools  
- Localhost apps  
- MCP clients  
  
PKCE = **Proof Key for Code Exchange**  
  
It protects against stolen authorization codes being misused.  
  
---  
  
## Step-by-Step Flow  
  
## 1. Client Generates PKCE Values  
  
The client creates:  
  
### `code_verifier`  
  
A secret random string.  
  
Example:  
  
super-secret-random-string  
  
### `code_challenge`  
  
A SHA256 hash of the verifier.  
  
Example:  
  
SHA256(code_verifier)  
  
This is sent to the authorization server.  
  
### Why?  
  
Later, the server verifies that:  
  
> The same client that started the flow is the one exchanging the authorization code.  
  
This prevents attackers from stealing the code and using it.  
  
---  
  
## 2. Redirect User to `/authorize`  
  
The client sends the browser to:  
  
`/oauth2/authorize`  
  
with query params like:  
  
- `client_id`  
- `redirect_uri`  
- `state`  
- `code_challenge`  
- `code_challenge_method=S256`  
  
Example:  
  
http://localhost:8000/oauth2/authorize?client_id=apifrenzy-mcp&redirect_uri=http://localhost:8000/callback&code_challenge=...&code_challenge_method=S256&state=teststate  
  
This means:  
  
> Authenticate the user and ask for consent.  
  
---  
  
## 3. User Logs In + Approves Access  
  
The authorization server handles:  
  
- login  
- authentication  
- consent screen  
- security checks  
  
The client app never sees the user’s password.  
  
This is a major security benefit.  
  
---  
  
## 4. Server Redirects Back with Authorization Code  
  
Example redirect:  
  
http://localhost:8000/callback?code=abc123&state=teststate  
  
The `code` is:  
  
- short-lived  
- single-use  
- not the actual access token  
  
Think of it like:  
  
> A temporary claim ticket  
  
not actual permission.  
  
---  
  
## 5. Client Exchanges Code at `/token`  
  
The client sends a POST request to:  
  
`POST /oauth2/token`  
  
with:  
  
- `grant_type=authorization_code`  
- `code`  
- `redirect_uri`  
- `client_id`  
- `code_verifier`  
  
Example:  
  
```json  
{  
"grant_type": "authorization_code",  
"code": "abc123",  
"client_id": "apifrenzy-mcp",  
"redirect_uri": "http://localhost:8000/callback",  
"code_verifier": "original-secret-string"  
}
```

The server verifies:

SHA256(code_verifier) == original code_challenge

If valid, it returns:

```json
```{  
  "access_token": "...",  
  "refresh_token": "...",  
  "expires_in": 3600  
}

```

---

## 6. Client Uses Access Token

The client now calls protected APIs using:

Authorization: Bearer ACCESS_TOKEN

This is the actual permission to access resources.

---

## Important Distinction: Code vs Token

Many people confuse these.

They are very different.

## Authorization Code

- temporary
- single-use
- short-lived
- must be exchanged for a token

Think:

> valet ticket

## Access Token

- used for API access
- grants actual permission
- expires later
- used in Authorization header

Think:

> actual car key

---

## Simple Analogy

### Authorization Code

Temporary valet ticket

### Access Token

Actual car key

### PKCE

Proof that the valet ticket belongs to you

---

## Why Not Return the Token Directly?

Because exposing tokens in browser redirects is dangerous.

Authorization codes are safer because:

- they are short-lived
- single-use
- require PKCE verification

This is why modern OAuth prefers Authorization Code Flow over Implicit Flow.
