# Quickstart

## Prerequisites

- Python 3.11+
- Nothing else for a paper run. Redis is optional (bar cache mirror)

## Install

```bash
git clone https://github.com/mevcel/pip
cd pip
python3.11 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pip install -e ".[dev]"
cp .env.example .env
```

Leave `.env` alone for now. With no trading key the agent is unarmed and
`PONS_PIP_MODE` defaults to `paper`.

## One cycle

```bash
pons-pip cycle
```

Ingest, signals, portfolio, risk gate, execute, once, then a JSON report:

```json
{
  "started_at": "2026-09-04T14:02:11Z",
  "pairs": 3,
  "bars": 3,
  "signals": 4,
  "orders": 1,
  "decisions": [{"order_id": "...", "action": "PASS"}],
  "fills": [{"pair_id": "PONS-AAPL", "side": "buy", "amount_usd": 250.0, "simulated": true}],
  "arm_state": "unarmed"
}
```
Fills in paper mode are simulated against the modeled quote and recorded with
`simulated: true`.

## Keep it running

```bash
pons-pip run
```

Cycles every `engine.cycle_interval_s` seconds (60 by default) until Ctrl+C.

## Replay a CSV

```bash
pons-pip replay --csv bars.csv
```

Columns: `pair_id, ts, open, high, low, close, volume_usd`. A replay is
always paper, whatever `.env` says. This is how you test a strategy change
before a PR.

## Look at it

```bash
pons-pip state          # JSON self-report
pons-pip disclosure     # same as text, with limits and the disclaimer
```

Prometheus is on `127.0.0.1:8000/metrics` while `pons-pip mcp` runs.

## Going live

Read [OPERATIONS.md](OPERATIONS.md) first. Short version: a dedicated wallet,
its key in `PONS_PIP_TRADING_KEY`, its address in `PONS_PIP_BUDGET_ADDRESS`,
`PONS_PIP_MODE=live`, a confirmed router address and ABI. All four or the
agent stays unarmed.
