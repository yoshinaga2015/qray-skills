---
name: qray
description: >-
  Create and manage QRay dynamic QR codes from an AI agent using the `qray` CLI
  (@qray/cli) or the hosted QRay MCP server. Use when the user wants to create a
  QR code for a URL, list their QR codes, change where an existing (possibly
  already printed) QR code points, pause or resume a QR code, download QR artwork
  (SVG/PNG), check scan counts, or organize QR codes into projects.
---

# QRay

QRay makes **dynamic QR codes**: the printed code stays the same, and you can change where it sends people at any time. This skill lets an agent manage a user's QRay account through the `qray` CLI or the hosted MCP server.

Using QRay from agents (CLI or MCP) requires credits on the user's QRay account. Credit purchase is coming soon.

## When to use

- Create a QR code for a URL, campaign, flyer, or event
- List the user's QR codes or look one up
- **Change the destination** of an existing QR code (often one that is already printed)
- **Pause** or resume a QR code
- Download QR artwork as SVG or PNG for print
- Check **scan counts** for one or several QR codes
- Organize QR codes into projects

## Setup

Both options use an **API key that the user creates in the QRay dashboard** (https://qray.link/dashboard). Never ask the user to paste the key into chat if they can set it up themselves; never print the key back.

### Option 1: CLI

```bash
npm i -g @qray/cli        # installs the `qray` command
# or run once without installing: npx @qray/cli <command>

qray auth login --key <API key from the dashboard>
qray auth status
```

Add `--json` to any command for machine-readable output.

### Option 2: MCP

QRay hosts an MCP server at `https://go.qray.link/mcp` (remote HTTP). Authenticate with the dashboard API key as a Bearer token in the `Authorization` header.

After `qray auth login`, the CLI can print a ready-to-paste config:

```bash
qray mcp config --client cursor
qray mcp config --client claude-code
qray mcp config --client claude-desktop
```

Paste the output into the client's MCP settings. Any MCP host that supports remote HTTP servers with a custom header can use the same URL and header.

MCP tool names mirror the CLI:

| Task | CLI | MCP tool |
|------|-----|----------|
| List QR codes | `qray qrs list` | `list_qrs` |
| Get one QR code | `qray qrs get <id>` | `get_qr` |
| Create | `qray qrs create --destination-url <url>` | `create_qr` |
| Update destination / name / memo / project | `qray qrs update <id> …` | `update_qr` |
| Pause / resume | `qray qrs pause <id>` / `resume <id>` | `pause_qr` / `resume_qr` |
| Delete / restore | `qray qrs delete <id> --confirm-slug <slug>` / `restore <id>` | `delete_qr` / `restore_qr` |
| Artwork | `qray qrs svg <id>` / `qray qrs png <id>` | `get_qr_svg` / `get_qr_png` |
| Scan stats | `qray qrs stats <id> [id…]` | `get_qr_stats` / `batch_qr_stats` |
| Projects | `qray projects list\|create\|update\|delete\|reorder` | `list_projects`, `create_project`, `update_project`, `delete_project`, `reorder_projects` |
| Credit balance | `qray balance` | `get_balance` |

Run `qray --help` for every flag.

## Safety rules (always follow)

1. **Confirm with the user before changing a destination URL or deleting** a QR code or project. Show the QR's name, current destination, and the new destination first.
2. **Printed QR codes should be paused, not deleted.** Deleting breaks every copy already in the field.
3. **Dry-run writes first.** Use `--dry-run` (CLI) or `dry_run: true` (MCP), show the result to the user, then run the real write.
4. **No bulk changes without explicit approval.** List the IDs and the intended change for each before touching many QR codes.
5. Treat the API key as a secret. Don't echo it, log it, or commit it.

## Credits

Most operations consume credits from the user's QRay account. Checking the balance and dry runs are free.

- Run `qray balance` / `get_balance` before loops or bulk work, and tell the user roughly what the batch will use.
- Prefer one `list` call over many `get` calls.
- Use multi-ID stats (`qray qrs stats id1 id2 …` / `batch_qr_stats`) instead of one call per QR.
- If credits run out, tell the user and point them to the dashboard. Don't retry in a loop.

## Recipes

### Create a QR code for a URL

```bash
qray qrs create --destination-url "https://example.com/launch" --name "Launch flyer" --dry-run
qray qrs create --destination-url "https://example.com/launch" --name "Launch flyer"
qray qrs png <id> -o launch-flyer.png
```

### Change where a printed QR code points

1. `qray qrs get <id>` and show the user the current destination.
2. `qray qrs update <id> --destination-url "https://example.com/new" --dry-run`
3. After the user confirms: run the same command without `--dry-run`.

### Temporarily stop a QR code

```bash
qray qrs pause <id>     # asks for confirmation unless --yes
qray qrs resume <id>
```

### Check scans

```bash
qray qrs stats <id>
qray qrs stats <id1> <id2> <id3> --json
```

### Organize into projects

```bash
qray projects list
qray projects create --name "Summer campaign"
qray qrs update <id> --project-id <projectId> --dry-run
```

## CLI behavior notes

- `qrs delete`, `qrs pause`, `qrs update --destination-url`, and `projects delete` ask for confirmation. Pass `--yes` only after the user has confirmed in the conversation. With `--json` in a non-interactive shell, these commands require `--yes`.
- `qrs delete` needs the QR's slug via `--confirm-slug` as an extra safety check. Get it from `qray qrs get <id>`.
- Exit codes: `0` ok, `2` usage, `3` auth, `4` not found, `5` not enough credits, `6` too many requests (wait and retry later), `7` validation, `1` other.

## Errors: what to tell the user

| Situation | What to do |
|-----------|-----------|
| Auth failed | The API key is missing, wrong, or revoked. Ask the user to create a new key in the dashboard. |
| Permission denied on a write | The key may be read-only. Ask the user to create a key with write access. |
| Not found | Wrong ID or the QR was deleted. Run `qrs list` to find it. |
| Not enough credits | Tell the user and link the dashboard. Don't retry. |
| Too many requests | Slow down and retry later. |
| Validation error | Show the message and fix the input (URL format, missing slug, etc.). |
| Conflict / locked QR | Explain it, and offer to pause instead of changing the destination. |

## Links

- Agent docs: https://qray.link/en/docs/agents (日本語: https://qray.link/ja/docs/agents)
- Dashboard (API keys, credits): https://qray.link/dashboard
- CLI on npm: https://www.npmjs.com/package/@qray/cli
