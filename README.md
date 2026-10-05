# OpenClawCash agent wallet skill

[![skills.sh](https://skills.sh/b/openclawcash/agentwalletapi)](https://skills.sh/openclawcash/agentwalletapi)

A crypto wallet for OpenClaw and every other AI agent. Your agent gets an **API key, never a private
key**. It can send, swap, bridge and trade on Ethereum, Polygon, Base and Solana, and **every action is
checked against the limits you set before anything is signed**.

This repo is the agent skill for [OpenClawCash](https://openclawcash.com): a plain `SKILL.md` in the
`agentskills.io` standard plus a curl-based CLI, so it works with any skill-capable agent. Also
referred to as `openclawcash`.

## Guardrails

Policies are set per wallet in the dashboard and enforced by OpenClawCash before signing. A request
that breaks one is rejected, logged, and never reaches the chain. The agent cannot change or bypass them.

| Guardrail | What it does |
|---|---|
| Per-transaction cap | Nothing above the amount you set goes out in a single transfer or swap. |
| Rolling limits | Daily, weekly and monthly limits, so many small requests can't add up to a big loss. |
| Address allowlist | Only send to addresses or @handles you've approved. |
| Testnet only | Let a new agent practise on Sepolia or Solana devnet first, for free. |
| Feature gates | Decide per wallet whether Polymarket or escrow checkout are allowed at all. |
| Every attempt recorded | API calls and onchain transactions side by side, including the blocked ones. |

The API key itself is scoped too: wallet creation, venues, checkout and live transactions are separate
permissions you switch on in the dashboard.

## What your agent can do

- **Send and receive** native coins plus ERC-20 and SPL tokens, by symbol or contract / mint address,
  in human-readable amounts. Every response returns the fee and the net amount.
- **Swap** on Uniswap (Ethereum, Polygon, Base) and Jupiter (Solana): quote, approve, swap.
- **Bridge** between EVM chains and Solana through LiFi, with a quote first and a status call after.
- **Trade on Polymarket** from a Polygon wallet: search markets, place market and limit orders,
  cancel, list positions, redeem winnings.
- **Get paid with escrow:** create a payment request, the buyer funds it, the seller submits proof,
  and the funds are released, refunded or disputed. Agents can also be paid by a handle such as
  `@studio-agent`.
- **Check before acting:** read balances, transaction history and the wallet's policies first.

Everything runs through the same policy check, whether the call comes from this skill, the MCP server
or the REST API. Full endpoint reference: [`references/api-endpoints.md`](references/api-endpoints.md).

## Install

**If your client supports MCP, use the MCP server.** Tools, schemas and results arrive structured:

```bash
npx -y @openclawcash/mcp-server@0.1.27
```

Otherwise install this skill:

```bash
# OpenClaw: tell your agent
#   "Visit https://openclawcash.com/agentwalletapi and install the OpenClawCash skill for me."

# Hermes Agent
hermes skills install openclawcash/agentwalletapi

# Claude Code and other clients: copy into the client's skills directory
git clone https://github.com/openclawcash/agentwalletapi ~/.claude/skills/agentwalletapi
```

Then create an API key at [openclawcash.com](https://openclawcash.com), run `bash scripts/setup.sh`
to create a `.env` in the skill folder, and put the key in it. Never commit that file.

| | |
|---|---|
| Required env var | `AGENTWALLETAPI_KEY` (your OpenClawCash API key) |
| Optional env var | `AGENTWALLETAPI_URL` (default `https://openclawcash.com`) |
| Required binary | `curl` (`jq` optional, for pretty output) |
| Network access | `https://openclawcash.com` |

## How the skill keeps the agent safe

- **No private keys, ever.** OpenClawCash holds the wallets with keys encrypted at rest. The agent only
  has an API key. Importing an existing wallet is done by you in the dashboard, never by the agent.
- **Writes need approval.** At the first write the agent asks you for a mode: approve every write, or
  approve once and let it act for the session inside your guardrails. The CLI requires `--yes` for
  every write.
- **Labels are data, not instructions.** Wallets are selected by ID for writes, and a destination,
  amount or token is never taken from a label.
- **The key goes to one host only.** The CLI sends `X-Agent-Key` only to `https://openclawcash.com`
  (or a subdomain) and refuses any other `AGENTWALLETAPI_URL`.

## Files

```
SKILL.md                     Skill instructions and workflow
scripts/setup.sh             Creates .env with your API key placeholder
scripts/agentwalletapi.sh    CLI for clients without MCP
references/api-endpoints.md  Full endpoint documentation
```

## License

MIT, see [LICENSE](LICENSE). "OpenClawCash" is a trademark of OpenClawCash; forks must not imply
endorsement.
