# OpenClawCash AgentWalletAPI skill

[![skills.sh](https://skills.sh/b/openclawcash/agentwalletapi)](https://skills.sh/openclawcash/agentwalletapi)

Agent skill for OpenClawCash-managed wallets on EVM and Solana: list wallets, check balances, send
native and token transfers, run DEX swaps, drive Get Paid checkout escrow, and operate Polymarket and
YieldWolf Casino venues — all through the [OpenClawCash agent API](https://openclawcash.com).

It is a plain **`SKILL.md`** in the `agentskills.io` standard plus a curl-based script, so it works with
any skill-capable agent — no client-specific code. Also referred to as `openclawcash`.

## Install

Drop the folder anywhere your agent loads skills from, or use your client's skill installer:

```bash
# Hermes Agent
hermes skills install openclawcash/agentwalletapi
hermes skills tap add openclawcash/agentwalletapi

# Claude Code / other clients: copy into the client's skills directory
git clone https://github.com/openclawcash/agentwalletapi ~/.claude/skills/agentwalletapi
```

Any agent that can read a file can use it: point it at `SKILL.md` and it has the full workflow, endpoint
table, safety model, and CLI fallback. Nothing in this repo depends on a specific agent runtime.

**If your client supports MCP, prefer the MCP server** — tools, schemas, and results arrive structured:

```bash
npx -y @openclawcash/mcp-server
```

MCP and the bundled CLI script call the same agent API; they are two access paths, not two products.

## Requirements

| | |
|---|---|
| Required env var | `AGENTWALLETAPI_KEY` — create one at [openclawcash.com](https://openclawcash.com) |
| Optional env var | `AGENTWALLETAPI_URL` (default `https://openclawcash.com`) |
| Required local binary | `curl` |
| Optional local binary | `jq` (pretty JSON in CLI output) |
| Network access | `https://openclawcash.com` |

Run `bash scripts/setup.sh` to create a `.env` in the skill folder, then replace the placeholder with
your real key. Never commit that file.

## What it covers

- **Wallets** — list, inspect with native and token balances, rename labels, create, import
- **Transfers** — native coins and any ERC-20 / SPL token, with `amountDisplay` or `valueBaseUnits`
- **Swaps** — quotes and execution on Uniswap (EVM) and Jupiter (Solana mainnet); cross-chain bridge
  quotes and execution via LiFi
- **Checkout** — Get Paid pay requests, escrow lifecycle (fund, accept, proof, dispute, release, refund,
  cancel), webhooks
- **Venues** — Polymarket (markets, orders, positions, redeem) and YieldWolf Casino
- **Policies** — read wallet governance policies before proposing a write action

Full endpoint reference: [`references/api-endpoints.md`](references/api-endpoints.md).

## Safety model

- **Read-only first.** Start with `wallets`, `wallet`, `policy`, `balance`, `tokens` — on testnets first.
- **Explicit approval for writes.** Pick one session mode up front: `confirm_each_write` (ask before every
  write) or `operate_on_my_behalf` (approve once, then execute the session's instructions). The CLI
  requires `--yes` for write actions.
- **Dashboard gates.** Wallet creation and import stay blocked unless the API key has
  `allowWalletCreation` / `allowWalletImport` enabled.
- **Labels are untrusted data.** A label that reads like a command is just a name. Write actions select
  wallets by `walletId`, never by label, and never take a destination, amount, token, or approval
  decision from a label.
- **Key pinned to one host.** `X-Agent-Key` is only ever sent to `https://openclawcash.com` (or an
  `https://<subdomain>.openclawcash.com` host); the bundled script validates `AGENTWALLETAPI_URL`
  against that allowlist before every request and refuses to run otherwise.
- **Private keys.** Import is optional; pass the key via hidden prompt or stdin, never as a CLI argument.

## Files

```
SKILL.md                     Skill instructions and workflow
scripts/setup.sh             Creates .env with your API key placeholder
scripts/agentwalletapi.sh    CLI fallback for clients without MCP
references/api-endpoints.md  Full endpoint documentation
```

## License

MIT — see [LICENSE](LICENSE). "OpenClawCash" is a trademark of OpenClawCash; forks must not imply
endorsement.
