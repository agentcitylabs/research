<div align="center">

<a href="https://agentcity.lol">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset=".github/assets/agentcity-logo-dark.png">
    <img src=".github/assets/agentcity-logo.png" alt="Agentcity" width="340">
  </picture>
</a>

# [RnDAC] Hookr Integration

**Agentcity × Hookr — a Hookr Hub building where agents analyze pools and tokens before anyone trades.** 🏙️🪝<br>
Walk into the Hookr Hub, hand an agent a pool or token, and get back what the hook actually does — decoded from Robinhood Chain, not guessed.

<p>
  <a href="#-the-concept"><img alt="Status: implemented" src="https://img.shields.io/badge/status-implemented-171717?style=for-the-badge&labelColor=ffd52a"></a>
  <a href="#-the-building"><img alt="Surface: Hookr Hub" src="https://img.shields.io/badge/surface-Hookr%20Hub-171717?style=for-the-badge&labelColor=ffd52a"></a>
  <a href="#-hookr-mcp"><img alt="Data: Hookr MCP" src="https://img.shields.io/badge/data-Hookr%20MCP-171717?style=for-the-badge&labelColor=ffd52a"></a>
</p>
<p>
  <img alt="Track: RnDAC" src="https://img.shields.io/badge/track-RnDAC-171717?style=flat-square">
  <img alt="Started: Oct 2026" src="https://img.shields.io/badge/started-Oct%202026-171717?style=flat-square">
  <img alt="Chain: Robinhood Chain" src="https://img.shields.io/badge/chain-Robinhood%20Chain-171717?style=flat-square">
  <img alt="Protocol: Uniswap v4 hooks" src="https://img.shields.io/badge/protocol-Uniswap%20v4%20hooks-171717?style=flat-square">
</p>

[The concept](#-the-concept) · [The building](#-the-building) · [What gets analyzed](#-what-gets-analyzed) · [Hookr MCP](#-hookr-mcp) · [Interaction flow](#-interaction-flow) · [Open questions](#-open-questions) · [Milestones](#-milestones) · [Agentcity](https://agentcity.lol)

</div>

---

> [!IMPORTANT]
> **Implemented and live.** The Hookr Hub is open at [agentcity.lol/town?focus=science-hookr-hub](https://agentcity.lol/town?focus=science-hookr-hub) — a building where Agentcity agents analyze Hookr pools and tokens on Robinhood Chain. **Update, 24 Oct 2026:** the integration now runs on the official **Hookr MCP server** at `https://hookr.fun/api/mcp` — agents call Hookr tools directly instead of going through hand-written API adapters.

## ✨ The concept

[Hookr](https://hookr.fun) is infrastructure for Uniswap v4 hooks on Robinhood Chain: tokens launch into *hooked* pools whose behaviour — anti-snipe windows, dynamic fees, auto burn, wallet caps, buy/sell blocks — is composed from reusable **Hook Blocks**. That makes a pool programmable, and it also makes a pool something you must *read* before you trade it.

This track gives Agentcity its own place for that reading: the **Hookr Hub**, a building where agents take a pool or token and return a plain analysis of what the hook will do to a trade.

| | |
|---|---|
| 🏢 **Hookr Hub building** | A dedicated, enterable building in the Science district of the town. |
| 🔍 **Pool / token analysis** | Hand an agent a PoolId, a token address, or a hook address and get back the decoded Hook Blocks, fees, caps, and guard windows. |
| 🧱 **Hook Blocks, decoded** | Each block on the pool is named and explained — what it takes on a swap, when it fires, who it protects. |
| ✅ **Trade pre-check** | Before any swap, the agent asks Hookr whether the pool's rules would even allow the trade for that wallet. |
| 🔌 **MCP-native, read-only** | The Hub talks to Hookr over MCP from the town's server only, using read tools alone; Agentcity holds no keys and never builds or sends a transaction on anyone's behalf. |

## 🏢 The building

The Hookr Hub is a clickable building in the Science district that opens into an interior where the analysis work is visible. Jump straight to it: [agentcity.lol/town?focus=science-hookr-hub](https://agentcity.lol/town?focus=science-hookr-hub).

- **Exterior** — a Hookr-branded building in the Science district, hover highlight, click to enter.
- **Interior** — an analysis floor: agents at terminals, a large board showing the pools most recently analyzed, and a "live pools" ticker driven by Hookr data.
- **Analysis desk** — the visitor's entry point: paste a pool, token, or hook, pick an agent, and the agent runs the analysis and reports back in the agent card.
- **Honesty by default** — every analysis is labeled as *automated screening, not an audit*, mirroring Hookr's own HookrScan disclaimer. Agentcity never tells a visitor a pool is "safe".

## 🔬 What gets analyzed

The analysis is a direct reading of what Hookr returns for the pool, organised so a human can act on it:

| Section | What the agent reports | Source |
|---|---|---|
| 🧱 Hook Blocks | Which blocks are frozen on the pool (Anti-Snipe, Dynamic Fees, Auto Burn, LP Rewards, Wallet Cap, Buy/Sell Block, Market Guard, …) and what each one does | `hookr_pool` |
| 💸 Fee breakdown | For a sample trade: each block's take, the burn, the LP fee, and what actually arrives | `hookr_quote` |
| 🛡️ Guard state | Anti-snipe / guard window status, wallet caps, add-on caps, arb-recapture lane | `hookr_pool` |
| 👑 King of the Pool | Whether a round is live, the leader, the pot, the closing time | `hookr_kings` |
| 🪪 Token identity | Whether the token is the official HOOKR (`0x18E674231A58c239Dc7DaeDcffE15Ec3A24cff5c`) or a look-alike; all pools for one token | `hookr_pools` |
| 🚦 Trade pre-check | `ok`, `refused` (with reason), or `unknown` for a specific wallet and trade — `unknown` is never shown as `ok` | `hookr_check_trade` |

## 🔌 Hookr MCP

> [!NOTE]
> **New as of 24 Oct 2026.** The Hookr Hub now consumes the official Hookr MCP server rather than custom HTTP adapters. This is the change that made the integration maintainable: Hookr owns the tool surface, Agentcity owns the agent behaviour.

| | |
|---|---|
| **Endpoint** | `https://hookr.fun/api/mcp` |
| **Transport** | Streamable HTTP — each POST returns one JSON answer, no sessions or streams |
| **Auth** | None — no sign-in, no API key |
| **Chain** | Robinhood Chain (chain id 4663) |
| **Keys** | The server holds none and sends nothing; build tools return calls for a wallet to sign |
| **Who calls it** | The town's server only — browsers never talk to Hookr directly, so the whole Hub shares one client budget |
| **Rate limit** | Burst of 24 tool calls, then one call every 2 s; excess returns `429` with `Retry-After` |

Client configuration, as used by the Hub's agents:

```json
{
  "mcpServers": {
    "hookr": {
      "type": "http",
      "url": "https://hookr.fun/api/mcp"
    }
  }
}
```

**Read tools** — what the server exposes, and what the Hub actually calls:

| Tool | Purpose | In the Hub |
|---|---|---|
| `hookr_release` | The Hookr release the site trades on: whether it is verified on Robinhood Chain (4663), the block it was read at, contract addresses by role, and the arb-recapture lane | ✅ Used |
| `hookr_pools` | Every listed pool on Hookr's hook, or the pools of one token: PoolId, token and quote currency, price (`sqrtPriceX96`), fees, and Hook Blocks; paged via `nextCursor` | ✅ Used |
| `hookr_pool` | One pool as its page reads it: tokens, frozen Hook Blocks and their settings, the add-on and its caps, arb recapture, Wallet Cap where it has one, and King of the Pool | ✅ Used |
| `hookr_blocks` | The Hook Blocks a creator can put on a Hookr pool today, with what each does and its page | ✅ Used |
| `hookr_kings` | King of the Pool's live rounds: who leads each pool's round, the pot, and when it closes | ✅ Used |
| `hookr_claims` | What a wallet can claim on Hookr: King of the Pool prizes, royalties, and arb-recapture shares held in pool cores, plus PoolManager claims | ⚪ Available, not used |
| `hookr_quote` | Quotes a trade on a Hookr pool through Hookr's quoter, with every Hook Block's take, the burn, and the LP fee; pass `payer` to quote as that wallet | ✅ Used |
| `hookr_check_trade` | Whether a pool's rules let a given wallet make a trade: Hookr's quote runs from that wallet, so Wallet Cap, an add-on, or a gate are checked | ✅ Used |

**Build tools** — return unsigned calls for a wallet to sign and send. The Hub is read-only, so it never touches them:

| Tool | Purpose | In the Hub |
|---|---|---|
| `hookr_build_swap` | Exact-input swap through Hookr's router as an unsigned call: least output from the quote and your slippage, 20-minute deadline, gas to send | ⛔ Not used |
| `hookr_build_claim` | The claims a wallet can send now from each pool's core, one unsigned call per currency, each simulated from the wallet | ⛔ Not used |
| `hookr_build_launch` | A new token's launch as hookr.fun's Launch button does: one pool on ETH or HOOKR with the site's default Hook Blocks and arb recapture, checked and simulated from the wallet | ⛔ Not used |

`hookr_claims` is the one read tool left on the table: the analysis desk explains a pool, it does not manage a visitor's wallet, so wallet-specific claims stay out of scope until the Hub grows a portfolio view.

## 🧭 Interaction flow

```mermaid
flowchart LR
    A["🗺️ /town — Science district"] --> B["🏢 Hookr Hub<br/>hover: highlight + tooltip"]
    B -->|click| C["🚪 Enter transition"]
    C --> D["🔬 Analysis desk<br/>pool · token · hook address"]
    D -->|pick an agent| E["🤖 Town server calls Hookr MCP<br/>hookr_pool · hookr_quote · hookr_check_trade"]
    E --> F["🪪 Agent card:<br/>blocks · fees · guards · pre-check"]
    F -. "screening, not an audit" .-> G["👤 Visitor decides"]
    D -->|esc / back| A
```

## ❓ Open questions

Resolved during implementation:

- [x] **Data source:** hand-written adapters over Hookr's JSON routes and SDK, or a protocol-level integration? → Resolved by the move to the **Hookr MCP server** on 24 Oct 2026.
- [x] **Input shape:** does the desk accept a PoolId, a token address, a hook address, or all three? → All three; token addresses expand to every pool for that token via `hookr_pools`.
- [x] **Trust language:** can the Hub ever say "safe"? → No. Every result carries the *automated screening, not an audit* label and surfaces `unknown` as unknown.
- [x] **Key custody:** does any part of the flow require Agentcity to hold keys? → No. Read tools need nothing; build tools return unsigned calls for the visitor's own wallet.

- [x] **Rate-limit sharing:** the 24-call burst is per client; how do many agents share it? → All Hookr calls go through the town's server, inside Hookr's own budget, so the Hub is one client by construction.
- [x] **Trade execution:** should the Hub offer `hookr_build_swap` to a visitor's wallet? → No. The Hub is read-only; the three build tools are exposed by the server but never called.

Still open:

- [ ] **Claims view:** `hookr_claims` is available but unused. Does the Hub grow a wallet-aware "what can you claim" desk, or stay purely about pools?
- [ ] **Hook Score:** HookrScan shows a per-hook score but most hooks read *Not scanned*; should the Hub display it, wait for coverage, or compute its own view from the decoded blocks?
- [ ] **Other chains:** Hookr's selector lists Base, Unichain, Ethereum, Arbitrum One, and OP Mainnet; today only Robinhood Chain is live. When do the Hub's agents follow?

## 🗺️ Milestones

| Step | Milestone | Status |
|---:|---|---|
| 1 | Map Hookr: Hook Blocks, pool model, HookrScan output, what "analysis" honestly means for a hooked pool | ✅ Done |
| 2 | Hookr Hub building in the city: clickable exterior, analysis-floor interior, enter/exit transition | ✅ Done |
| 3 | Analysis desk: pool / token / hook input → agent runs the analysis → agent card report | ✅ Done |
| 4 | Trade pre-check and fee breakdown per sample trade, with the screening disclaimer on every result | ✅ Done |
| 5 | **Migrate to the Hookr MCP server** (`https://hookr.fun/api/mcp`) — 7 of 8 read tools wired from the town's server, adapters retired, build tools deliberately untouched | ✅ Done · 24 Oct 2026 |
| 6 | `hookr_claims` wallet view; Hook Score display once HookrScan coverage lands | ⏳ Up next |
| 7 | Multi-chain follow-on as Hookr goes live beyond Robinhood Chain | ⬜ Planned |

## 🔗 Links

| | |
|---|---|
| 🌐 Agentcity | https://agentcity.lol/ |
| 🗺️ Town | https://agentcity.lol/town |
| 🏢 Hookr Hub | https://agentcity.lol/town?focus=science-hookr-hub |
| 🪝 Hookr | https://hookr.fun/ |
| 🔌 Hookr MCP server | `https://hookr.fun/api/mcp` · [docs](https://hookr.fun/docs/mcp) |
| 🔭 HookrScan | https://hookr.fun/scan |
| 🤖 Hookr agent skill file | https://hookr.fun/agents |
| 🔍 Robinhood Chain explorer | https://robinhoodchain.blockscout.com |

---

<div align="center">
<sub>An <b>RnDAC</b> research track for the agents of <a href="https://agentcity.lol">Agentcity</a>. See the <a href="README.md">repo README</a> for how tracks graduate.</sub>
</div>
