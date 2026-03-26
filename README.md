# SpawnDock

<p align="center">
    <img width="900" src="https://raw.githubusercontent.com/SpawnDock/agent/refs/heads/main/docs/ai-hackathon-2026-submission.png" alt="box3301 logo" />
</p>

Open-source platform for building products on top of Telegram Mini Apps and the TON blockchain. SpawnDock provides the infrastructure layer — project scaffolding, live preview tunnels, and an AI knowledge base with 55+ documents on TMA and TON — so product teams can focus on what they're building, not how to set it up.

SpawnDock is not a single product. It's the foundation that different products are built on.

## TMA Spawner

The first product on the platform. [**@TMASpawnerBot**](https://t.me/TMASpawnerBot) is a Telegram bot that gives anyone — even without development experience — a way to create and preview a Telegram Mini App. Write `/new`, follow the instructions, and your app is live on a real device in under a minute.

The bot handles all the technical setup behind the scenes: cloning the template, configuring the tunnel, wiring up the AI assistant. The user just sees the result.

## Getting Started

- [**@TMASpawnerBot**](https://t.me/TMASpawnerBot) — start here if you want to create a Mini App
- [**GitHub**](https://github.com/SpawnDock) — source code and packages
- [**Web3 Voice**](https://t.me/w3voice) — community

## Platform Components

**Project Setup**

- [`@spawn-dock/create`](https://github.com/SpawnDock/create-spawn-dock) — Bootstrap CLI. Clones the starter template, configures the tunnel and MCP, links the project to the user's account.
- [`tma-project`](https://github.com/SpawnDock/tma-project) — Starter template. Next.js + TypeScript + TON Connect + Telegram UI.

**Development**

- [`@spawn-dock/dev-tunnel`](https://github.com/SpawnDock/dev-tunnel) — Tunnel client. Exposes `localhost` to Telegram for real-device preview without deployment.
- [`@spawn-dock/mcp`](https://github.com/SpawnDock/mcp-client) — MCP knowledge client. Gives AI agents (Claude, Cursor, Codex) access to 55+ docs on TMA, TON, wallets, smart contracts, and deployment.
- [`@spawn-dock/cli`](https://github.com/SpawnDock/cli) — AI runtime launcher. Starts the configured agent in a sandboxed environment inside the project.

## Knowledge Base

The MCP server provides AI agents with searchable access to **55+ documents** (29,500+ lines):

| Area                   | Coverage                                                                          |
|:-----------------------|:----------------------------------------------------------------------------------|
| **Telegram Mini Apps** | WebApp API, navigation, theming, testing, security, performance                   |
| **TON Blockchain**     | Smart contracts (Tolk / Tact / FunC), jettons, NFTs, DeFi, wallets, DNS, payments |
| **TON Connect**        | Wallet integration, authentication, TON Proof                                     |
| **Deployment**         | Cloudflare Pages, Vercel, GitHub Pages                                            |
| **Templates**          | Shop, game, landing, quiz, menu, portfolio                                        |

## License

MIT
