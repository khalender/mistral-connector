# doslim-agent

Chat agent that turns documents into doslim **worksheets** and **books**.

- Talks to doslim through the Studio MCP connector `doslim_86e9`, which must point at
  `https://doslim.app/mcp/signed-in` with OAuth 2.0 (see "Connector" below).
- Fetches the current skill file with `doslim_get_guide` before writing any JSON
  (one skill file per kind: `worksheet`, `book`; languages en/nl/de/fr, default en).
- Validates every file with `doslim_check_draft` until it reports `ok`.
- Submits with `doslim_add_draft` (fresh `requestId` + `guideVersion`), which files
  the draft in the **signed-in adult's** doslim account — the account and family
  space come from the OAuth token server-side, never from chat input.
- Uses a Mistral sandbox so uploaded documents can be read as source material.

## Connector

The connector address is `https://doslim.app/mcp/signed-in`, not `/mcp`. The agents
runtime signs the user in to a connector only before a session starts, and only when
the connector has OAuth 2.0 as its authentication method. doslim's `/mcp` answers
without a token (ChatGPT needs that), so a connector on `/mcp` is stored without
OAuth: guide and check work, and every `doslim_add_draft` fails with "Sign in to
doSlim to add drafts." `/mcp/signed-in` answers 401 with `WWW-Authenticate` to any
request without a token, so the connector is stored with OAuth and the adult is asked
to sign in when the chat starts. doslim registers the client at `/oauth/register`.

To fix an existing connector: in Mistral's console, set `doslim_86e9` to the
`/mcp/signed-in` address with OAuth 2.0 (or create it again with that address), then
redeploy the agent.

## Run

```bash
uv sync
export MISTRAL_API_KEY=...        # or put it in .env
uv run agents chat
```

## Deploy (Mistral-managed)

```bash
uv run agents deploy
```

The agent uses the doslim connector on behalf of the session user, so the
deployment is hardened by default; authorize the principal with a pin if
registration asks for it.

## Notes

- On `/mcp/signed-in` every tool needs the adult's OAuth sign-in (PKCE), which
  Mistral asks for when the chat starts. On `/mcp` only `doslim_add_draft` does,
  and it signals that in a way only ChatGPT reads.
- Guides currently served by doslim.app: worksheet `5d5c6f6144e4`, book
  `4543078674a3`. The agent always re-fetches them, so guide updates are picked
  up automatically.
