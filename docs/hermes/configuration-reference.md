# Hermes Configuration Reference

## Title
Hermes — Configuration Reference

## Purpose
This document defines all configuration parameters available for the Hermes service. All parameters are supplied via environment variables to ensure secrets-free configuration and 12-factor compliance.

## Status
Draft

## Version
0.1.0

## Last Updated
2026-06-25

## Author
mrmtsuruya

---

## Table of Contents

1. [Configuration Philosophy](#configuration-philosophy)
2. [Environment Variables](#environment-variables)
3. [Configuration File](#configuration-file)
4. [Secrets Management](#secrets-management)
5. [Example .env](#example-env)

---

## Configuration Philosophy

All configuration is supplied via environment variables. No secrets are stored in source code or configuration files. A `.env.example` file documents available variables; actual `.env` files are gitignored.

---

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `HERMES_HOST` | No | `0.0.0.0` | Service bind host |
| `HERMES_PORT` | No | `8000` | Service bind port |
| `HERMES_LOG_LEVEL` | No | `INFO` | Logging level |
| `WEBHOOK_SECRET` | ✅ | — | Secret for webhook authentication |
| `DATABASE_URL` | ✅ | — | Database connection string |
| `MT5_BRIDGE_URL` | ✅ | — | MT5 bridge service URL |
| `AI_MODEL_PATH` | No | — | Path to AI model weights |
| `ENVIRONMENT` | No | `development` | Runtime environment (`development`, `production`) |

---

## Configuration File

_[YAML or TOML config file support (if any) to be defined.]_

---

## Secrets Management

_[Integration with secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault) to be defined for production.]_

---

## Example .env

```dotenv
HERMES_HOST=0.0.0.0
HERMES_PORT=8000
HERMES_LOG_LEVEL=INFO
WEBHOOK_SECRET=change-me
DATABASE_URL=******localhost:5432/mdtsignal
MT5_BRIDGE_URL=http://localhost:9000
ENVIRONMENT=development
```
