# Deployment

## Docker

```bash
cp .env.example .env
docker compose up --build -d
docker compose logs -f pons-pip
```

Ports bind to `127.0.0.1` on the host. Secrets come from `.env`. The image
has no secrets baked in and starts unarmed in paper mode unless the
environment says otherwise.

## Systemd

`/etc/systemd/system/pons-pip.service`:

```ini
[Unit]
Description=PIP trading agent
After=network.target

[Service]
User=pons
WorkingDirectory=/opt/pons-pip
ExecStart=/opt/pons-pip/.venv/bin/pons-pip run
Restart=on-failure
RestartSec=10
EnvironmentFile=/opt/pons-pip/.env

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable pons-pip
systemctl start pons-pip
journalctl -u pons-pip -f
```

A restart comes back in whatever state it was in: a disarmed agent stays
disarmed, a tripped breaker stays tripped.

## Monitoring

Prometheus scrapes `127.0.0.1:8000/metrics` while `pons-pip mcp` is running.
Put an authenticating proxy in front of it before exposing it anywhere.

Alerts worth having:

- `pons_pip_drawdown_pct` above 1.5 (the breaker fires at 2)
- rate of `pons_pip_risk_blocks_total` above 10/min
- `pons_pip_feed_age_seconds` above 900 during market hours
- `pons_pip_breaker_tripped` at 1 for longer than you expect a person to
  take to look
- `pons_pip_armed` changing value at all

## Before arming

1. Confirm every `TODO(pons)` in `config/pons.yaml` against ponsfamily.com
2. Run `pons-pip state` and read it
3. Run `pons-pip cycle` once and read the report
4. Fund the budget wallet with only what the agent may lose, plus the gas
   reserve
5. Set the webhook so every decision reaches you
