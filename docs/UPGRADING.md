# Upgrading from upstream

This repo (`phillipadsmith/second-brain`) is a **clean mirror** of
`rahilp/second-brain-cloudflare`, currently synced to upstream **v2.0.5** as of
2026-07-22 (upstream commit `b1e11f0`).

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

That's it. The Worker auto-migrates its D1 schema on the first request after deploy
(gated by an in-memory `dbReady` flag, so it only runs once per isolate).

Before merging a large upstream gap (multiple minor/major versions), it's worth
sanity-checking scope first: `git log --oneline main..upstream/main` and
`git diff --stat main upstream/main` show what's actually changing. For anything
beyond a routine patch bump, merge into a throwaway branch first, run `npm test` +
`npm run typecheck` there, and only fast-forward `main` once it looks good.

## Notes / gotchas

- **As of the v2.0.5 sync, `npm run dev` / `npm run deploy` are plain `wrangler dev` /
  `wrangler deploy`.** The old `scripts/prepare-wrangler.mjs` workaround (injecting the
  Vectorize binding into a generated `wrangler.deploy.jsonc` at deploy time, to dodge a
  blank-fields bug in the Cloudflare one-click-deploy form) is gone — upstream now
  declares `vectorize` directly in `wrangler.jsonc`, and since this repo deploys via the
  CLI (not the one-click form), the bug it worked around never applied here anyway.
- **`wrangler dev` may fail to boot locally** with `Uncaught TypeError: Incorrect type
  for map entry '<NAME>': the provided value is not of type 'function or
  ExportedHandler'.` This is a local-only workerd validation quirk in some wrangler
  versions — it strictly checks every top-level named export of `src/index.ts` (several
  plain constants are exported alongside the default handler) and errors on the first
  non-handler one it finds. It does **not** affect `wrangler deploy --dry-run` or the
  actual deployed Worker (same code runs fine in production); it only blocks the local
  dev server on affected wrangler versions. If you hit it, try pinning an older/newer
  wrangler patch version.
- `AUTH_TOKEN` is stored as a Cloudflare secret and persists across deploys. Re-set it only
  interactively: `npx wrangler secret put AUTH_TOKEN`.
- Sanity check after deploy:
  ```sh
  curl -s -o /dev/null -w "%{http_code}\n" https://second-brain.phillipadsmith.workers.dev/      # 200
  curl -s -o /dev/null -w "%{http_code}\n" https://second-brain.phillipadsmith.workers.dev/mcp   # 401
  curl -s -o /dev/null -w "%{http_code}\n" https://second-brain.phillipadsmith.workers.dev/health # 401
  ```
- To confirm a schema migration actually landed on the live database:
  ```sh
  npx wrangler d1 execute second-brain-db --remote --command \
    "SELECT name FROM sqlite_master WHERE type='table';"
  ```
- Upstream also ships a Tauri desktop installer app under `installer/` (ecosystem hub,
  one-click Cloudflare provisioning, in-app updates). This repo pulls its source along
  with merges for parity with upstream, but it isn't built or used here — deployment is
  still the manual CLI flow above.

## Rollback

Before a risky upgrade, tag `main` first (e.g. `git tag backup-pre-<name>`) so you can
get back with one command:
```sh
git reset --hard backup-pre-<name> && npm install && npm run deploy
```
The pre-v1.9.0 state is preserved as the `backup-pre-v190` tag/branch, and the
pre-v2.0.5 state as the `backup-pre-v2` tag (local only unless pushed to origin).
