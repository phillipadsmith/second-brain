# Upgrading from upstream

This repo (`phillipadsmith/second-brain`) is a **clean mirror** of
`rahilp/second-brain-cloudflare`, re-baselined onto upstream **v1.9.0** on 2026-06-24.

> Note: this is NOT a GitHub fork and Cloudflare is NOT git-connected. So the upstream
> "Sync fork → Update branch" button and any auto-redeploy do not apply. Upgrade and
> deploy manually with the steps below.

## One-time setup (already done)

```sh
git remote add upstream https://github.com/rahilp/second-brain-cloudflare.git
```

## To upgrade to a new upstream release

```sh
git fetch upstream
git merge upstream/main          # or: git rebase upstream/main
# Resolve the only expected conflict: wrangler.jsonc — keep our pinned resource IDs
#   D1     database_id : 4f475851-3a09-4cb4-b6b6-b4b867fdf899
#   OAUTH_KV id        : 1b92783aa6544c2b9dda13f59b7e1977
git push origin main

npm install
npm run deploy
```

That's it. The Worker auto-migrates its D1 schema on the first request after deploy.

## Notes / gotchas

- **Deploy with `npm run deploy`, never plain `wrangler deploy`.** The Vectorize binding is
  intentionally kept out of `wrangler.jsonc`; `scripts/prepare-wrangler.mjs` creates the
  index and injects the binding into a generated `wrangler.deploy.jsonc` at deploy time.
- The `vectorize.index.duplicate_name [code: 3002]` error during deploy is **expected and
  harmless** — the index already exists; the script catches it and continues.
- `AUTH_TOKEN` is stored as a Cloudflare secret and persists across deploys. Re-set it only
  interactively: `npx wrangler secret put AUTH_TOKEN`.
- Sanity check after deploy:
  ```sh
  curl -s -o /dev/null -w "%{http_code}\n" https://second-brain.phillipadsmith.workers.dev/      # 200
  curl -s -o /dev/null -w "%{http_code}\n" https://second-brain.phillipadsmith.workers.dev/mcp   # 401
  ```

## Rollback

The pre-upgrade state is preserved as the `backup-pre-v190` tag and branch (on origin too).
To roll back: `git reset --hard backup-pre-v190 && npm install && npm run deploy`.
