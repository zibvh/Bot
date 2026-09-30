# NovaBot — Read-Only WhatsApp Dashboard

Personal WhatsApp Web automation bot with a live **read-only** browser dashboard.

## What it does

ONE WhatsApp Web connection powers both:
- 🤖 NovaBot commands (`.games`, `.rps`, `.dice`, `.8ball`, `.joke`, `.sticker`, `.ai`, group tools, etc.)
- 🌐 A browser dashboard that mirrors chats/messages live.

The dashboard intentionally has no WhatsApp send, read/seen, delete, react, archive, or typing controls. Opening/scrolling the dashboard is local UI activity; it does not intentionally call WhatsApp read APIs.

> This uses unofficial `whatsapp-web.js` automation of WhatsApp Web, not the official WhatsApp Business API. It is intended for personal experimentation and may break or be restricted if WhatsApp changes its systems.

## Local setup

1. Use Node.js 20+.
2. Run `npm install`.
3. Copy `.env.example` to `.env`.
4. Set `OWNER_NUMBER` if you want owner info in the bot. `DASHBOARD_TOKEN` is not required by v7.
5. Run `npm start`.
6. Open `/` and enter your WhatsApp number in international format (numbers only), then use the pairing code shown on the page.
7. Open `/dashboard`.

## Render

This repository includes `Dockerfile` and `render.yaml`.

Create a Render **Web Service** from the repository. The included configuration uses a persistent disk mounted at `/var/data`.

The WhatsApp session is stored at:
`/var/data/.wwebjs_auth`

Dashboard data is stored at:
`/var/data/data`

Keep the persistent disk: without persistent storage, the WhatsApp Web session can be lost on restarts/redeploys.

### Environment variables

- `BOT_NAME` — display name
- `PREFIX` — command prefix, normally `.`
- `OWNER_NUMBER` — your WhatsApp number, digits only with country code
- `AI_ENABLED` — `true`/`false`
- `OPENAI_API_KEY` — optional
- `OPENAI_MODEL` — AI model
- `AI_IN_GROUPS` — allow AI in groups
- `RESPOND_IN_GROUPS` — allow bot commands in groups
- `PORT` — Render web port
- `AUTH_PATH` — `/var/data/.wwebjs_auth`
- `DATA_PATH` — `/var/data/data`
- `DASHBOARD_TOKEN` — not used by v7; the connection page and dashboard are directly accessible.

`render.yaml` supplies the non-secret values. You do not need a dashboard token for v7.

### After deployment

Open:
- `/` — status
- `/health` — health JSON
- `/` — phone-number connection page
- `/dashboard` — read-only dashboard

If the session is not already authenticated, use the phone-number pairing flow from `/`.

## Commands

General:
`.help` `.menu` `.ping` `.alive` `.owner` `.about` `.time` `.echo hello` `.whoami`

Games:
`.games` `.rps rock` `.dice` `.coin` `.8ball question` `.guess`

Fun:
`.joke` `.quote` `.choose pizza|burger|rice` `.truth` `.dare` `.calc 12*8`

Groups:
`.groupinfo` `.admins` `.tagall` `.everyone` `.hidetag` `.link`

Media:
Reply to media with `.sticker`

AI:
`.ai question` `.resetai` (AI must be enabled in the server environment)

## Architecture

WhatsApp → whatsapp-web.js → { Bot engine + Socket.IO → Read-only dashboard }

The dashboard consumes events from the same running process and does not expose WhatsApp mutation controls.

## Security

Treat `.env`, the WhatsApp auth directory, Render persistent disk, and dashboard token as sensitive. Do not commit them to GitHub.

Avoid spam, bulk messaging, unsolicited automation, or abusive use.

## Troubleshooting

If pairing does not work, open `/` and generate a fresh phone-number pairing code. The connection page is phone-number pairing only.

If the session disappears after redeploy, verify the persistent disk is mounted at `/var/data` and `AUTH_PATH=/var/data/.wwebjs_auth`.

If the dashboard is empty, confirm WhatsApp reports `ready`, then refresh `/dashboard`.

## Connection page

The root page is now a dedicated WhatsApp connection page, separate from the read-only dashboard. It supports:

- **Pairing-code pairing:** enter your WhatsApp number in international format with country code, then enter the generated code in WhatsApp's linked-device flow. `whatsapp-web.js` currently exposes `requestPairingCode()` for this flow.
- The connection page and dashboard use the same underlying WhatsApp Web session.
- The connection page is not token-gated in v7, so you will not get an `Unauthorized` screen when opening the Render URL.

After deployment, open the service root (for example `/` with the token query parameter) for connection, and `/dashboard` for the separate read-only dashboard.


## NovaBot v7 connection mode
This build uses phone-number pairing only. The connection page has no QR button, tab, image, or /qr route. For a first-time unlinked Render instance, the WhatsApp browser is not started until you submit your phone number. If you still see an old QR screen after deploying this build, you are viewing an older Render deployment/repository build; hard-refresh the page and verify the page says `NovaBot v7` and `PHONE-NUMBER PAIRING ONLY`.
