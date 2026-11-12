# QRay Agent Skill

An [Agent Skill](https://skills.sh) that lets AI coding agents create and manage [QRay](https://qray.link) dynamic QR codes: create a QR for a URL, change where a printed QR points, pause it, download SVG/PNG artwork, and check scan counts.

It uses the `qray` CLI ([@qray/cli](https://www.npmjs.com/package/@qray/cli)) or QRay's hosted MCP server.

## Install

```bash
npx skills add yoshinaga2015/qray-skills
```

## Requirements

- A QRay account with purchased credits (agent access via CLI/MCP uses credits)
- An API key created in the [QRay dashboard](https://qray.link/dashboard)
- Node.js, for the `qray` CLI

## Quick start

```bash
npm i -g @qray/cli
qray auth login --key <API key from the dashboard>
qray qrs list
```

Then ask your agent things like:

- "Make a QR code for https://example.com/event and save it as a PNG."
- "Point the flyer QR to the new landing page." (the agent dry-runs and asks before changing it)
- "How many scans did my campaign QR codes get?"

## What's inside

- [`skills/qray/SKILL.md`](skills/qray/SKILL.md) — setup, safety rules, and recipes for the agent

## Links

- Docs for agents: https://qray.link/en/docs/agents
- QRay: https://qray.link
