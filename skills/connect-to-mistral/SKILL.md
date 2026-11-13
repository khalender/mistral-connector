---
name: connect-to-mistral-button
description: Add a "Connect to Mistral" button to the doslim app that sends an adult from doslim.app to the doslim-chat agent on Mistral, using the adult's own Mistral account. Use when adding the Mistral integration entry point to doslim's settings or dashboard.
---

# Add the "Connect to Mistral" button to doslim

You add one button to doslim, an app for families. The button sends an adult
(parent or teacher) from doslim to a chat agent hosted by Mistral. That agent
("doslim-chat") turns documents into doslim worksheets and books and adds them
as drafts to the adult's doslim account. You write no agent code, store no
tokens and call no Mistral API: the button is a link, and Mistral's own pages
handle the Mistral sign-in.

## 1. The rules

- **Adults only.** Show the button only to signed-in adult accounts. Children
  never see it.
- **No keys anywhere in doslim.** Never embed a Mistral API key, client secret
  or token in doslim's frontend, backend or config for this feature. The adult
  authenticates with their own Mistral account on Mistral's site; doslim never
  sees it.
- **The doslim account comes from the token, never from the URL.** When the
  agent later adds a draft, doslim's MCP server derives the account and family
  space from the OAuth access token alone. Never accept an account id, space id
  or user id for this flow from the browser, a query parameter or the agent.
- **Do not change the draft flow.** The button only opens the chat. Draft
  creation, checks, requestId rules and the "no photos" rule stay exactly as
  they are in the MCP server.

## 2. The button

- Label: `Connect to Mistral`.
- Placement: the adult's settings page or dashboard, wherever integrations live.
- Behavior: opens the agent's canonical chat URL in a new tab.
- Get the URL by copying it, never by building it: open chat.mistral.ai, go to
  "All agents", open `doslim-chat` the normal way, and copy the full URL from
  the browser's address bar once the chat is open. That URL is the canonical
  deep link (it names the agent by its id, not by its name). A URL constructed
  from the agent's name, such as
  `https://chat.mistral.ai/managed-agents/doslim-chat?source=agents_api`, can
  answer "Page not found"; do not use one.
- Store the copied URL as one constant, for example `MISTRAL_AGENT_URL`, in the
  frontend config; do not build it from parts. If the agent is ever redeployed
  under a new id, re-copy the URL and update the constant.
- Helper text under the button, one line, for example:
  "Opens a chat by Mistral. Sign in with your own Mistral account, then allow
  doslim once when asked. Drafts land in your doslim for review."
- No callback to doslim is needed after the click: doslim cannot and should not
  read the Mistral session. Do not add a return URL, listener or state check
  for the click itself.

## 3. What the adult sees after the click (for your copy and support text)

1. Mistral asks them to sign in with their own Mistral account, if they are not
   already. This is Mistral's page, not doslim's.
2. The doslim-chat page opens. The agent asks what to make (worksheet or
   book), the grade, the language (en, nl, de or fr) and for documents.
3. The first time the agent adds a draft, Mistral shows a consent prompt for
   the doslim connector. The adult allows it, and doslim's own OAuth consent
   page opens ("doslim asks to work with your doSlim"), signed in with the
   adult's doslim email and password. After Allow, the browser returns to the
   chat and the agent finishes.
4. The agent replies with the draft's `reviewUrl`. The draft sits in the
   adult's doslim space, origin as configured, until the adult reviews it.

## 4. One-time server check before shipping

The connector sign-in uses doslim's existing OAuth server (PKCE, S256). Verify
that it accepts Mistral's connector client the same way it accepts ChatGPT's:

- The well-known endpoints (`/.well-known/oauth-protected-resource`,
  `/.well-known/oauth-authorization-server`) answer for any client.
- Whatever client allow-list or client metadata check exists (the ChatGPT flow
  fetches a `client.json` to verify the client) also admits Mistral's connector
  client, or the server registers it dynamically.
- The consent page shows the client's real name, and Allow issues a code bound
  to the adult's account and family space, exactly as for ChatGPT.

Test this once end to end: from doslim, click the button, sign in on Mistral,
ask for a small worksheet, allow the connector, and confirm the draft appears
in the adult's doslim with a working `reviewUrl`.

## 5. Check before you finish

1. The button is hidden for child accounts and shown for adults.
2. The URL constant matches the URL in section 2, query parameter included.
3. No Mistral key, secret or token appears anywhere in the doslim codebase,
   config or logs for this feature.
4. No doslim code passes an account or space id into this flow.
5. The one-time server check in section 4 passed, and a test draft is visible
   in the adult's doslim.
