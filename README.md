# Redline — Frontend

Static frontend for [Redline](https://redline.fourallthedogs.com), an AI-powered car marketplace. Deployed to S3 and served globally via CloudFront with WAF protection and a wildcard ACM certificate.

No build step required — pure HTML, CSS, and JavaScript.

---

## Pages

| File | Description |
|------|-------------|
| `index.html` | Homepage — lists all available vehicles |
| `listing.html` | Individual vehicle detail page with negotiation form |
| `negotiate.html` | Negotiation result page (deal reached or failed) |
| `messages.html` | Full negotiation history for the authenticated user |
| `login.html` | Cognito hosted UI redirect and auth callback handler |

---

## Auth Flow

Uses the Amazon Cognito Identity JS SDK. On sign-in, Cognito issues a JWT stored by the SDK. All pages check session state via `userPool.getCurrentUser()` — no raw localStorage token handling.

Protected pages (`messages.html`, `negotiate.html`) redirect unauthenticated users to `login.html`.

---

## Negotiation Flow

1. User browses listings on `index.html`
2. Clicks a listing → `listing.html` loads vehicle details from the listings API
3. Authenticated user enters their max budget and submits
4. `listing.html` POSTs to `/negotiate/start` — all negotiation rounds run server-side synchronously via Bedrock
5. On completion the user is redirected to `negotiate.html` with the deal outcome
6. Full conversation history is available on `messages.html`

---

## Infrastructure

- **Hosting:** S3 static website
- **CDN:** CloudFront with WAF WebACL and wildcard ACM certificate
- **DNS:** `redline.fourallthedogs.com` via Route53

All infrastructure managed in [redline-terraform](https://github.com/djiwani/redline-terraform).

---

## CI/CD

GitHub Actions (`.github/workflows/deploy-frontend.yml`) triggers on every push to `main`:

1. Syncs all files to S3 with `--delete` to remove stale files
2. Creates a CloudFront invalidation on `/*` to purge the cache

Changes are live within approximately 30 seconds.

---

## Manual Deployment

```bash
# Sync to S3
aws s3 sync . s3://redline-frontend \
  --exclude ".git/*" \
  --exclude ".github/*" \
  --exclude "README.md" \
  --delete \
  --region us-east-1 \
  --profile dev

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1FYHMQIX40Q8P \
  --paths "/*" \
  --profile dev
```

---

## Related Repositories

- [redline-terraform](https://github.com/djiwani/redline-terraform) — All AWS infrastructure
- [redline-api](https://github.com/djiwani/redline-api) — FastAPI microservices
