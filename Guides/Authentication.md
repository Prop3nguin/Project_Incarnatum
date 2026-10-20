---
description: How to secure Acme's API with API keys or OAuth 2.0, including key generation, scopes, provider configuration, role mapping, and session management.
---

# Authentication

When running Acme's API server, you'll want to secure access. Acme supports API keys for simple setups and OAuth 2.0 for production environments.

## API keys

The simplest authentication method. Generate a key and include it in requests.

### Generate a key

```bash
acme auth create-key --name "CI Pipeline" --scope "pipelines:read,pipelines:run"
```

```
Created API key:
  Name:  CI Pipeline
  Key:   df_key_a1b2c3d4e5f6...
  Scope: pipelines:read, pipelines:run

Store this key securely — it won't be shown again.
```

### Using API keys

```bash
# HTTP header
curl -H "Authorization: Bearer df_key_a1b2c3d4e5f6..." \
  https://acme.example.com/api/v1/pipelines

# Query parameter (not recommended)
curl "https://acme.example.com/api/v1/pipelines?api_key=df_key_..."
```

> [!danger] API key security
>
> - Never commit API keys to version control
> - Rotate keys regularly with `acme auth rotate-key`
> - Use the minimum required scope for each key
> - Prefer OAuth 2.0 for production environments

### Available scopes

| Scope             | Description                          |
| ----------------- | ------------------------------------ |
| `pipelines:read`  | View pipeline definitions and status |
| `pipelines:run`   | Trigger pipeline runs                |
| `pipelines:write` | Create, update, delete pipelines     |
| `admin`           | Full access to all resources         |

## OAuth 2.0

For production environments, Acme supports OAuth 2.0 with your identity provider.

### Configuration

```yaml
# acme.yml
auth:
  provider: oauth2
  issuer: https://auth.example.com
  client_id: ${OAUTH_CLIENT_ID}
  client_secret: ${OAUTH_CLIENT_SECRET}
  audience: https://acme.example.com
  scopes:
    - openid
    - profile
    - email
```

### OAuth flow

```mermaid
sequenceDiagram
    participant U as User
    participant DF as Acme API
    participant IDP as Identity Provider

    U->>DF: Request /api/v1/pipelines
    DF-->>U: 401 Unauthorized
    U->>IDP: Authenticate
    IDP-->>U: Access Token (JWT)
    U->>DF: Request + Bearer Token
    DF->>IDP: Validate Token
    IDP-->>DF: Token Valid + Claims
    DF-->>U: 200 OK + Pipeline data
```

### Supported providers

- **Okta** — `provider: okta`
- **Auth0** — `provider: auth0`
- **Google Workspace** — `provider: google`
- **Azure AD** — `provider: azure`
- **Generic OIDC** — `provider: oauth2`

### Role mapping

Map OAuth claims to Acme roles:

```yaml
auth:
  role_mapping:
    admin:
      claim: groups
      values: ["acme-admins"]
    editor:
      claim: groups
      values: ["acme-editors", "data-team"]
    viewer:
      claim: groups
      values: ["acme-viewers", "engineering"]
```

> [!note]
> Role mappings are evaluated in order. A user matching multiple roles gets the highest-privilege role.

## Session management

For the web dashboard, Acme uses secure cookies:

```yaml
auth:
  session:
    secret: ${SESSION_SECRET}
    max_age: 86400 # 24 hours
    secure: true # HTTPS only
    same_site: strict
```

## Related

- [[API Reference/Client API|Client API]] — making authenticated API calls
- [[Guides/Deployment|Deployment]] — production security configuration
- [[Configuration/Environment Variables|Environment Variables]] — storing secrets
