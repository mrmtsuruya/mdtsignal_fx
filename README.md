# mdtsignal_fx

> **Institutional-Grade AI-Assisted Trading Platform — XAUUSD & BTCUSD**

---

## Overview

**mdtsignal_fx** is a professional-grade, AI-assisted trading platform engineered for high-precision signal generation, risk management, and execution across XAUUSD (Gold/USD) and BTCUSD (Bitcoin/USD) markets.

The platform integrates:
- **TradingView** — Pine Script signal automation and webhook relay
- **Hermes** — Core Python signal engine & AI decision layer
- **Dashboard** — Real-time monitoring and performance analytics
- **MetaTrader 5 (MT5)** — Automated trade execution via MQL5 bridge
- **Database** — Time-series signal, trade, and audit storage

---

## Repository Structure

```
mdtsignal_fx/
├── .github/            # CI/CD workflows and issue templates
├── config/             # Environment and runtime configuration
├── dashboard/          # Front-end monitoring dashboard
├── data/               # Raw and processed market data
├── database/           # Schema, migrations, and seed data
├── docs/               # Full project documentation
│   ├── ITS/            # Integrated Trading System specs
│   ├── api/            # API reference documentation
│   ├── engines/        # Signal engine architecture docs
│   ├── hermes/         # Hermes service documentation
│   └── research/       # Research notes and backtesting reports
├── hermes/             # Core AI signal engine (Python)
├── mt5/                # MetaTrader 5 bridge (MQL5 / Python)
├── tests/              # Unit, integration, and e2e tests
├── tools/              # Developer utilities and scripts
└── tradingview/        # Pine Script strategies and alerts
```

---

## Key Documents

| Document | Description |
|---|---|
| [ROADMAP.md](ROADMAP.md) | Milestones and feature timeline |
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guidelines |
| [docs/ITS/](docs/ITS/) | Integrated Trading System specification |
| [docs/engines/](docs/engines/) | Signal engine architecture |
| [docs/api/](docs/api/) | API reference |

---

## Status

| Component | Status |
|---|---|
| Repository Initialization | ✅ Complete |
| Hermes Engine | 🔲 Planned |
| TradingView Integration | 🔲 Planned |
| MT5 Bridge | 🔲 Planned |
| Dashboard | 🔲 Planned |

---

## License

See [LICENSE](LICENSE) for details.

---

*Author: mrmtsuruya — Last Updated: 2026-06-25*
