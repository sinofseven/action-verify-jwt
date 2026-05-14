# action-verify-jwt

A GitHub Action that verifies a JWT token using a JWKS endpoint. Useful for validating OIDC tokens issued by identity providers.

> **Note:** This action supports Linux x86_64 runners only (`ubuntu-*`). macOS and Windows runners are not supported.

## Inputs

| Name | Required | Description |
|------|----------|-------------|
| `token` | Yes | JWT token to verify |
| `jwks-url` | Yes | URL of the JWKS endpoint (e.g., `https://token.actions.githubusercontent.com/.well-known/jwks`) |
| `audience` | No | Expected audience value. If provided, the action fails when the `aud` claim does not match |

## Usage

### Basic — verify signature only

```yaml
- uses: sinofseven/action-verify-jwt@v1.0.0
  with:
    token: ${{ steps.get-token.outputs.token }}
    jwks-url: https://token.actions.githubusercontent.com/.well-known/jwks
```

### With audience validation

```yaml
- uses: sinofseven/action-verify-jwt@v1.0.0
  with:
    token: ${{ steps.get-token.outputs.token }}
    jwks-url: https://token.actions.githubusercontent.com/.well-known/jwks
    audience: https://my-service.example.com
```

### Verifying a GitHub Actions OIDC token

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
    steps:
      - name: Get OIDC token
        id: get-token
        run: |
          TOKEN=$(curl -sS -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
            "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=my-service" | jq -r .value)
          echo "token=$TOKEN" >> "$GITHUB_OUTPUT"

      - name: Verify token
        uses: sinofseven/action-verify-jwt@v1.0.0
        with:
          token: ${{ steps.get-token.outputs.token }}
          jwks-url: https://token.actions.githubusercontent.com/.well-known/jwks
          audience: my-service
```

## How It Works

1. Downloads [jwt-cli](https://github.com/mike-engel/jwt-cli) and [oidc-jwks-converter](https://github.com/sinofseven/oidc-jwks-converter)
2. Fetches public keys from the JWKS URL and converts them to PEM format
3. Extracts the `kid` (Key ID) from the JWT header
4. Verifies the JWT signature using the matching public key
5. Optionally validates the `aud` (audience) claim

## License

[MIT](LICENSE)
