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
4. Set `OWNER_NUMBER` and a strong `DASHBOARD_TOKEN`.
5. Run `npm start`.
6. Open `/qr` and scan it from WhatsApp → Settings → Linked devices → Link a device.
7. Open `/dashboard?token=YOUR_TOKEN`.

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
- `DASHBOARD_TOKEN` — secret token for dashboard access

`render.yaml` already supplies the non-secret values and can generate `DASHBOARD_TOKEN`.

### After deployment

Open:
- `/` — status
- `/health` — health JSON
- `/qr` — QR/connection page
- `/dashboard?token=YOUR_TOKEN` — read-only dashboard

Scan the QR with your phone if the session is not already authenticated.

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

If the QR does not appear, check `/qr` and Render logs. An already-authenticated session may not show a QR.

If the session disappears after redeploy, verify the persistent disk is mounted at `/var/data` and `AUTH_PATH=/var/data/.wwebjs_auth`.

If the dashboard is empty, confirm WhatsApp reports `ready`, then refresh `/dashboard`.

## Connection page

The root page is now a dedicated WhatsApp connection page, separate from the read-only dashboard. It supports:

- **QR code pairing:** scan the QR shown on the connection page from WhatsApp → Settings → Linked devices → Link a device.
- **Pairing-code pairing:** enter your WhatsApp number in international format with country code, then enter the generated code in WhatsApp's linked-device flow. `whatsapp-web.js` currently exposes `requestPairingCode()` for this flow.
- The connection page and dashboard use the same underlying WhatsApp Web session.
- The connection page is protected by `DASHBOARD_TOKEN` just like the dashboard, so the QR/pairing controls are not public.

After deployment, open the service root (for example `/` with the token query parameter) for connection, and `/dashboard` for the separate read-only dashboard.
