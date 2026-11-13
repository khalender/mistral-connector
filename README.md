# doslim-agent

Chat agent that turns documents into doslim **worksheets** and **books**.

- Talks to doslim through the Studio MCP connector `doslim_86e9` (`https://doslim.app/mcp`).
- Fetches the current skill file with `doslim_get_guide` before writing any JSON
  (one skill file per kind: `worksheet`, `book`; languages en/nl/de/fr, default en).
- Validates every file with `doslim_check_draft` until it reports `ok`.
- Submits with `doslim_add_draft` (fresh `requestId` + `guideVersion`), which files
  the draft in the **signed-in adult's** doslim account — the account and family
  space come from the OAuth token server-side, never from chat input.
- Uses a Mistral sandbox so uploaded documents can be read as source material.

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

- `doslim_get_guide` and `doslim_check_draft` need no sign-in; only
  `doslim_add_draft` requires the adult's OAuth sign-in (PKCE, handled by the
  connector's sign-in flow).
- Guides currently served by doslim.app: worksheet `5d5c6f6144e4`, book
  `4543078674a3`. The agent always re-fetches them, so guide updates are picked
  up automatically.
