# DRAFT — wiki addition for "Connect to AI Clients"

> Not part of the PR. Paste/adapt this into the **Connect-to-AI-Clients** wiki page
> (https://github.com/rahilp/second-brain-cloudflare/wiki/Connect-to-AI-Clients) once the
> OAuth PR is merged. The existing Bearer-token instructions for Claude Desktop / Claude
> Code / `mcp-remote` stay exactly as they are — this only adds the browser-OAuth path.

---

## claude.ai and ChatGPT (OAuth)

Browser-based clients like **claude.ai** and **ChatGPT** authenticate over OAuth 2.0 — no
token in the URL. You enter your `AUTH_TOKEN` once on a hosted login page and the client
stores the resulting OAuth token.

### claude.ai

1. Go to **Settings → Connectors → Add custom connector**.
2. **Name:** `Second Brain`
3. **URL:** `https://<your-worker-url>/mcp`
4. Click **Add**, then **Connect**. A *Second Brain* login page opens.
5. Enter your **`AUTH_TOKEN`** (the same token you set during deploy) and click **Connect**.
6. You’ll be redirected back and the connector shows as connected. The `remember`,
   `recall`, `append`, and other tools are now available in chat.

### ChatGPT

1. In a Custom GPT or via **Settings → Connectors**, add a new MCP server.
2. **URL:** `https://<your-worker-url>/mcp`
3. Choose **OAuth** as the authentication method.
4. Complete the flow — when the *Second Brain* login page appears, enter your
   **`AUTH_TOKEN`** and click **Connect**.

> **Tested:** Claude desktop + iOS, ChatGPT desktop + iOS.

### Troubleshooting

- **Login page rejects the token** — make sure you’re entering your `AUTH_TOKEN`, not a
  worker URL or password. It’s the same value used in the `Authorization: Bearer` header.
- **Stuck or looping OAuth flow** — the client may have cached a bad state from an earlier
  attempt. Remove the connector and re-add it to force a clean flow.
- **`/oauth/authorize` shows "Invalid authorization request"** — that page is only meant
  to be opened by an MCP client mid-flow, not visited directly.

---

## Which auth path do I use?

| Client | Auth |
|--------|------|
| Claude Desktop, Claude Code, `mcp-remote` | `Authorization: Bearer <AUTH_TOKEN>` header (unchanged) |
| claude.ai, ChatGPT (and other browser MCP clients) | OAuth — enter `AUTH_TOKEN` on the login page |

Both paths use the same `AUTH_TOKEN`; there’s no separate password to manage.
