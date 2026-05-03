# Redline — Frontend

Static frontend for [Redline](https://redline.fourallthedogs.com), an AI-powered car marketplace. Deployed to S3 and served via CloudFront with WAF protection and a wildcard ACM certificate.

## Pages

| File | Description |
|------|-------------|
| `index.html` | Homepage — lists all available vehicles |
| `listing.html` | Individual vehicle detail page with negotiation form |
| `negotiate.html` | Negotiation result page (deal reached or failed) |
| `messages.html` | Full negotiation history for the authenticated user |
| `login.html` | Cognito hosted UI redirect and auth callback handler |

## Auth Flow

Authentication uses the **Amazon Cognito Identity JS SDK**. On sign-in, Cognito issues a JWT stored in the browser via the SDK. All pages read the current session via `userPool.getCurrentUser()` — no raw localStorage token handling.

Protected pages (messages, negotiation) redirect unauthenticated users to the login page.

## Negotiation Flow (Frontend)

1. User browses listings on `index.html`
2. Clicks a listing → `listing.html` loads vehicle details from the listings API
3. Authenticated user enters their max budget and hits **Negotiate Now**
4. `listing.html` POSTs to `/negotiate/start` — the API runs all negotiation rounds synchronously
5. On completion, user is redirected to `negotiate.html` with the outcome
6. User can view full conversation history on `messages.html`

## Infrastructure

- **Hosting**: S3 (`redline-frontend` bucket, static website)
- **CDN**: CloudFront distribution `E1FYHMQIX40Q8P`
- **Security**: WAF WebACL on CloudFront, wildcard ACM cert
- **DNS**: `redline.fourallthedogs.com` via Route53

## CI/CD

GitHub Actions workflow (`.github/workflows/deploy-frontend.yml`) triggers on every push to `main`:

1. Syncs all files to S3 (`aws s3 sync`) with `--delete` to remove stale files
2. Creates a CloudFront invalidation on `/*` to purge the cache

No build step required — pure HTML/CSS/JS, no bundler.

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

## Related Repositories

- [redline-terraform](https://github.com/djiwani/redline-terraform) — All infrastructure
- [redline-api](https://github.com/djiwani/redline-api) — FastAPI microservices
