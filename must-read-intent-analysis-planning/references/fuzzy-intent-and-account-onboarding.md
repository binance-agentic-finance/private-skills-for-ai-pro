# Fuzzy intent and account onboarding

## Description

For vague first messages or broad intent (e.g. “what can you do”, “how do I start as a newbie”, “teach me to make money”, “what features do you have”, “hello”).

The agent should:

1. **First** pull and interpret the user’s **current asset state** (`assets.getUserAssets`) so you know whether they can support **trading, earn, bots**, etc.
2. Give a **concise** overview of the main capability areas (trading, earn, trading bots, etc.).
3. Based on whether funds are sufficient, suggest **next steps**: empty account → **proactively** guide deposit, P2P/fiat buy, or internal transfer (“cold start”); funded → guide first trade or earn.

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

**Immediately** call **`assets`** (`getUserAssets`): distribution across wallets; focus **`SPOT`** and **`FUNDING`** totals and non-zero assets. Decide whether **available funds** support trading / earn / bots they might want.

- **If empty or clearly insufficient** for their implied goal: **Proactively tell them** (no tradable balance, need deposit or P2P/fiat) **before** long feature tours—offer concrete next actions. Optional **`p2p`** / **`fiat`** context for on-ramps.

### §A Structured pipeline (DAG)

**Step 2: Icebreaker and capability overview**

- Short bullet overview:
  1. **Smart trading and analysis** (spot/futures orders, market snapshot, klines)
  2. **Wealth products** (principal-protected earn, staking-style products)
  3. **Automation** (grid, DCA-style bots)
- Transition: *“To tailor advice, I’ve checked your balance—here’s what fits…”* (only after Step 1).

**Step 3: Conditional guidance**

- **A: Empty or very low balance**  
  - State clearly there is no meaningful tradable balance.  
  - **On-ramp**: crypto deposit address; fiat/P2P options (optional `p2p` for quote context).  
  - Example: *“Your account looks empty. To earn or trade we need funds first—deposit from another wallet, or buy with card/P2P?”*

- **B: Funds idle (e.g. all in Funding, or USDT idle)**  
  - Suggest internal **transfer** to spot/futures for trading, or introduce Simple Earn for idle stablecoins.

- **C: Spot funded with holdings**  
  - Higher-level ideas: grids/DCAs for stables; for BTC/ETH etc., brief market context or mention TP/SL—**still compliant, not guaranteed returns**.

### §B Endpoint quick reference

- **Balances**: `assets` → `getUserAssets`.
- **Buy channels (optional)**: `p2p` → e.g. ads/quote endpoints per **`p2p`** skill docs for motivation-only context.
