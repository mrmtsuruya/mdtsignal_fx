# Institutional Trading Specification — ITS v1.0

## Title
mdtsignal_fx — Institutional Trading Specification (ITS)

## Purpose
This document is the constitutional and architectural foundation of the mdtsignal_fx platform. It establishes the project vision, governing principles, technical standards, mathematical philosophy, risk hierarchy, AI hierarchy, and module responsibilities that all contributors and all system components must adhere to. It is the supreme reference document of the project and supersedes all lower-level design documents in the event of conflict.

## Status
Active

## Version
1.0.0

## Last Updated
2026-06-25

## Author
mrmtsuruya

---

## Table of Contents

1. [Project Vision](#1-project-vision)
2. [System Architecture](#2-system-architecture)
3. [The Platform Constitution](#3-the-platform-constitution)
4. [Technical Standards](#4-technical-standards)
5. [Mathematical Philosophy](#5-mathematical-philosophy)
6. [Evidence-First Philosophy](#6-evidence-first-philosophy)
7. [Versioning Rules](#7-versioning-rules)
8. [Coding Standards](#8-coding-standards)
9. [Performance Requirements](#9-performance-requirements)
10. [Non-Repainting Requirements](#10-non-repainting-requirements)
11. [Risk Hierarchy](#11-risk-hierarchy)
12. [AI Hierarchy](#12-ai-hierarchy)
13. [Module Responsibilities](#13-module-responsibilities)
14. [Amendment Process](#14-amendment-process)

---

## 1. Project Vision

### 1.1 Mission Statement

mdtsignal_fx is an institutional-grade, AI-assisted trading platform engineered to deliver **precision, reproducibility, and risk-controlled execution** across two flagship instruments: **XAUUSD (Gold / USD)** and **BTCUSD (Bitcoin / USD)**.

The platform exists to transform high-quality quantitative signals into disciplined trade execution — with every decision auditable, every risk bounded, and every outcome measurable.

### 1.2 Design Objectives

| Objective | Standard |
|---|---|
| **Precision** | Signal accuracy ≥ 60% win rate on statistically significant sample (n ≥ 200 trades per instrument) |
| **Repeatability** | Identical inputs must produce identical outputs across all environments |
| **Auditability** | Every signal, decision, and order must be permanently logged with full provenance |
| **Risk Discipline** | No single trade may risk more than the configured maximum risk-per-trade; system halts before breaching maximum drawdown |
| **Transparency** | All AI model decisions must be explainable and logged; no black-box execution |
| **Resilience** | The platform must degrade gracefully; failure of one component must not cascade into uncontrolled position exposure |

### 1.3 Target Operating Environment

The platform is designed to operate in institutional and semi-institutional contexts:

- **Instruments:** XAUUSD, BTCUSD
- **Execution venue:** MetaTrader 5 (MT5) compatible broker
- **Signal source:** TradingView Pine Script strategies via HTTPS webhook
- **Deployment:** Cloud-hosted services, containerized, with persistent database backing
- **Operating hours:** 24/5 (XAUUSD) and 24/7 (BTCUSD)

### 1.4 Non-Goals

The following are explicitly out of scope for this platform:

- High-frequency trading (sub-second execution)
- Market-making or arbitrage strategies
- Portfolio management across multiple asset classes
- Retail-facing brokerage services
- Autonomous strategy discovery without human oversight

---

## 2. System Architecture

### 2.1 Architecture Principles

The mdtsignal_fx architecture is governed by the following mandatory principles:

| Principle | Requirement |
|---|---|
| **Separation of Concerns** | Each module has a single, well-defined responsibility. No module may assume responsibilities assigned to another. |
| **Stateless Processing** | The Hermes signal engine is stateless per request. State is owned exclusively by the database layer. |
| **Event-Driven Design** | All inter-component communication is event-driven. Components consume and produce structured events; they do not share memory. |
| **Defense in Depth** | Risk controls are enforced at multiple layers independently: signal validation, Hermes AI gating, and MT5 execution checks. |
| **Immutable Audit Trail** | All signal and order events are written to an append-only audit log. No event record may be deleted or modified after creation. |
| **Configuration Externalization** | All runtime configuration is supplied via environment variables. Zero secrets in source code or configuration files. |

### 2.2 Component Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SIGNAL SOURCES                               │
│                                                                     │
│   TradingView Pine Script                                           │
│   (XAUUSD Strategy │ BTCUSD Strategy)                               │
│           │                                                         │
│           │  HTTPS Webhook (TLS 1.3)                                │
└───────────┼─────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       HERMES ENGINE                                 │
│                                                                     │
│  ┌─────────────┐   ┌──────────────────┐   ┌─────────────────────┐  │
│  │  Receiver   │──▶│  Signal Pipeline │──▶│   AI Decision Layer │  │
│  │ (Webhook    │   │  (Validate →     │   │  (Confidence Score  │  │
│  │  Listener)  │   │   Normalize →    │   │   Market Regime     │  │
│  └─────────────┘   │   Enrich)        │   │   Gate/Pass)        │  │
│                    └──────────────────┘   └─────────────────────┘  │
│                                                   │                 │
└───────────────────────────────────────────────────┼─────────────────┘
                                                    │
            ┌───────────────────────────────────────┤
            │                                       │
            ▼                                       ▼
┌───────────────────────┐             ┌─────────────────────────────┐
│      MT5 BRIDGE       │             │         DATABASE            │
│                       │             │                             │
│  Order Management     │             │  Signal Events (append-only)│
│  Position Sizing      │             │  Trade Records              │
│  Risk Pre-checks      │             │  Audit Log                  │
│  Execution            │             │  Performance Metrics        │
└───────────────────────┘             └─────────────────────────────┘
            │
            ▼
┌───────────────────────┐
│     BROKER (MT5)      │
│  Live / Paper Account │
└───────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        DASHBOARD                                    │
│  Real-time signal feed │ Position monitor │ PnL │ System health     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.3 Data Flow Specification

| Step | From | To | Protocol | Payload Format |
|---|---|---|---|---|
| 1 | TradingView | Hermes Receiver | HTTPS POST | JSON (Webhook Payload Schema v1) |
| 2 | Hermes Receiver | Signal Pipeline | Internal function call | Signal Domain Object |
| 3 | Signal Pipeline | AI Decision Layer | Internal function call | Validated Signal + Market Context |
| 4 | AI Decision Layer | MT5 Bridge | HTTP/gRPC (TBD) | Approved Signal Command |
| 5 | Hermes Engine | Database | Database driver | Signal Event Record |
| 6 | MT5 Bridge | MT5 Terminal | MT5 API | Order Request |
| 7 | MT5 Bridge | Database | Database driver | Order/Trade Record |
| 8 | Database | Dashboard | WebSocket / REST | Aggregated State |

### 2.4 Deployment Architecture

- All services are containerized (Docker).
- Services communicate over an isolated private network.
- The Hermes webhook endpoint is the only public-facing surface.
- The dashboard is accessible via authenticated reverse proxy only.
- The MT5 bridge operates within the private network; it is never directly exposed.

---

## 3. The Platform Constitution

The Platform Constitution is a set of inviolable rules. No feature request, performance optimization, or deadline pressure may override any Constitutional rule. Any proposed change to a Constitutional rule requires a formal amendment (see Section 14).

### 3.1 The Eight Constitutional Rules

**Rule I — Capital Preservation Is Supreme**
The primary function of the risk system is to preserve trading capital. Profit generation is secondary. Any signal, however confident, that would breach a configured risk limit **must** be rejected before reaching execution.

**Rule II — No Blind Execution**
No order may be sent to a broker without a complete, validated signal record in the database. Every execution must have full provenance: which strategy generated the signal, what the AI scored it, what the risk pre-check approved, and what parameters were used.

**Rule III — Non-Repainting Is Non-Negotiable**
All signal logic must be computed on confirmed, closed bars only. Any signal derived from an unclosed bar is architecturally prohibited. This rule applies to both Pine Script strategies and any derived signal re-computation performed in Hermes. See Section 10 for enforcement details.

**Rule IV — Fail Closed, Never Open**
In any failure mode — network error, service crash, invalid data, timeout — the system's default behaviour is to take no action. The platform **must never** open a new position during a degraded state. It **may** close existing positions defensively at the operator's discretion, but opening is forbidden.

**Rule V — Human Override Is Always Available**
Every automated action may be overridden by a human operator in real time. The platform must expose a control interface that allows an operator to halt all new signal processing, close all positions, or disable any module independently, without requiring a code deployment.

**Rule VI — Evidence Before Logic**
No strategy logic may be promoted to production without passing the defined evidence gates (see Section 6). Intuition, backtests alone, or theoretical reasoning are not sufficient for production deployment.

**Rule VII — Every Parameter Is Configured, Never Hard-Coded**
All operationally relevant parameters (risk percentages, lot sizes, confidence thresholds, drawdown limits, instrument-specific settings) must be sourced from the configuration system at runtime. Hard-coded values in logic are a critical defect.

**Rule VIII — The Audit Log Is Sacred**
The audit log is append-only and immutable. No process — including superuser database access — may delete, modify, or truncate audit records. Any architectural change that would impair the integrity of the audit log is prohibited.

---

## 4. Technical Standards

### 4.1 Language Standards

| Language | Minimum Version | Usage |
|---|---|---|
| Python | 3.11 | Hermes engine, MT5 bridge connector, tools |
| MQL5 | Current (MetaEditor) | MT5 Expert Advisor (execution side only) |
| Pine Script | v5 | TradingView signal strategies |
| SQL | PostgreSQL 15+ | Database schemas and queries |
| YAML | 1.2 | CI/CD configuration |

### 4.2 Dependency Management

- Python dependencies are managed via `pyproject.toml` (PEP 621) with version pinning in `requirements.lock`.
- All dependencies are scanned against known vulnerability databases before addition.
- Transitive dependencies must be explicitly pinned in lock files.
- No dependency may be added to production code without documented justification.

### 4.3 Interface Standards

| Interface | Standard |
|---|---|
| HTTP API | RESTful, JSON, OpenAPI 3.1 specification required |
| Webhook Authentication | HMAC-SHA256 signature verification |
| Database | Connection pooling required; raw SQL preferred over full ORM for performance-critical paths |
| Configuration | Environment variables, `.env.example` always in sync with actual variables |
| Logging | Structured JSON logging (key-value pairs); human-readable log level on development, JSON on production |

### 4.4 Security Standards

- All external communication uses TLS 1.3 minimum.
- Webhook endpoints authenticate every incoming request via HMAC-SHA256 signature.
- No credentials, API keys, or tokens in source control.
- Secrets are injected at runtime via environment variables or a secrets manager.
- Database connections use parameterized queries exclusively; string interpolation into SQL is a critical defect.
- Principle of least privilege applies to all service accounts and database roles.

### 4.5 Observability Standards

Every service must implement:

| Telemetry | Requirement |
|---|---|
| **Health endpoint** | `GET /health` returning service status, version, and uptime |
| **Structured logs** | JSON format, including `timestamp`, `level`, `service`, `correlation_id`, `message` |
| **Metrics** | Signal throughput, processing latency (p50/p95/p99), error rates, active positions |
| **Correlation ID** | Every signal must carry a `correlation_id` that propagates through all downstream events |
| **Alerting** | Critical errors and risk limit breaches must trigger operator alerts within 30 seconds |

---

## 5. Mathematical Philosophy

### 5.1 Core Principles

The mdtsignal_fx platform is built on the following mathematical and statistical principles. Violation of these principles in strategy design or model development is a defect requiring remediation before production deployment.

**Principle 1 — Stationarity Awareness**
Financial time series are non-stationary. All models and strategies must account for regime changes and avoid the assumption that historical statistical properties persist indefinitely. Models must be retrained or recalibrated on a defined schedule.

**Principle 2 — Expected Value Orientation**
Every signal and strategy must be evaluated on **expected value (EV)**, not win rate alone. A strategy with a 40% win rate and 3:1 reward-to-risk ratio is strictly superior to a 70% win rate strategy with 0.5:1 reward-to-risk ratio. EV = (Win Rate × Average Win) − (Loss Rate × Average Loss).

**Principle 3 — Sample Size Discipline**
Statistical conclusions require adequate sample sizes. The minimum accepted sample size for any performance claim is **200 completed trades per instrument per timeframe**. Claims derived from fewer observations are inadmissible as evidence for production promotion.

**Principle 4 — Out-of-Sample Primacy**
In-sample performance is hypothesis generation, not evidence. All strategy evaluation for production promotion must be conducted on **out-of-sample** data that the strategy has never encountered during development. Walk-forward analysis is the preferred validation methodology.

**Principle 5 — Distribution Awareness**
Trade outcomes do not follow a normal distribution. Fat tails, skewness, and kurtosis must be explicitly measured and accounted for in risk sizing. Value-at-Risk (VaR) calculations must use empirical distributions or fat-tail models (e.g., Cornish-Fisher expansion), never the normal assumption.

**Principle 6 — Volatility-Adjusted Sizing**
Position sizing must be adaptive to current market volatility. Fixed lot sizes are prohibited for live trading. The position sizing model must incorporate a volatility measure (e.g., ATR-based sizing) that scales exposure inversely with volatility.

**Principle 7 — Correlation Accounting**
When both XAUUSD and BTCUSD positions are open simultaneously, their correlation must be measured and factored into total risk exposure. The system must not treat concurrent positions as independent when they share meaningful correlation.

### 5.2 Required Statistical Metrics

The following metrics are mandatory outputs of any strategy evaluation:

| Metric | Formula / Definition | Minimum Acceptable Threshold |
|---|---|---|
| Win Rate | Winning trades / Total trades | ≥ 40% (EV-positive strategies may fall below) |
| Profit Factor | Gross Profit / Gross Loss | ≥ 1.5 |
| Sharpe Ratio | (Mean Return − Risk-Free Rate) / Std Dev of Returns | ≥ 1.0 (annualized) |
| Max Drawdown | Max peak-to-trough equity decline | ≤ 20% |
| Calmar Ratio | CAGR / Max Drawdown | ≥ 1.0 |
| Expectancy per Trade | EV = (WR × Avg Win) − (LR × Avg Loss) | > 0 (strictly positive) |
| R-Multiple Distribution | Distribution of trade outcomes in units of initial risk | Median R ≥ 0.5 |

---

## 6. Evidence-First Philosophy

### 6.1 The Evidence Gate

No strategy, model, or signal logic may be promoted to production unless it has passed through all four evidence gates in sequence. Gates may not be skipped.

```
Gate 1: Theoretical Validity
        │
        ▼
Gate 2: In-Sample Backtesting
        │
        ▼
Gate 3: Out-of-Sample Validation
        │
        ▼
Gate 4: Paper Trading (Live Data, No Real Capital)
        │
        ▼
     PRODUCTION
```

### 6.2 Gate Definitions

**Gate 1 — Theoretical Validity**
- A written hypothesis explains *why* the edge should exist in market microstructure or macro dynamics.
- The hypothesis identifies the market condition (regime) in which the edge is expected to be present.
- The hypothesis identifies the market condition in which the strategy is expected to fail.
- A domain expert (quantitative analyst or experienced trader) reviews and approves the hypothesis.

**Gate 2 — In-Sample Backtesting**
- Minimum 3 years of historical data.
- Minimum 200 completed trades.
- All required statistical metrics (Section 5.2) must meet or exceed their thresholds.
- Backtesting must use realistic spread and slippage assumptions.
- Walk-forward segmentation must be defined before backtesting begins (no post-hoc train/test splits).

**Gate 3 — Out-of-Sample Validation**
- Conducted on data strictly excluded from Gate 2.
- Minimum 6 months of out-of-sample data.
- Minimum 50 completed trades in the out-of-sample period.
- Performance degradation of more than 40% relative to in-sample metrics is a Gate failure and requires hypothesis revision.

**Gate 4 — Paper Trading**
- Signals run live in the full production stack against real market prices, but no real capital is committed.
- Minimum duration: 4 weeks of active trading.
- Minimum 20 paper trades completed.
- All execution latency, webhook reliability, and AI scoring behaviour are monitored and logged.
- Any system failures, signal gaps, or anomalies must be investigated and resolved before production promotion.

### 6.3 Evidence Documentation Requirements

For each gate, the following must be produced and archived:

- A written report with full statistical analysis.
- The configuration (parameters, settings) used during evaluation.
- The data range used, with source and version.
- The name of the reviewer who approved the gate passage.
- The date of approval.

These documents are stored in `docs/research/` and are referenced in the CHANGELOG at the time of production promotion.

---

## 7. Versioning Rules

### 7.1 Semantic Versioning

All software components follow [Semantic Versioning 2.0.0](https://semver.org/): `MAJOR.MINOR.PATCH`

| Increment | When to Use |
|---|---|
| **MAJOR** | Breaking change to any external interface, signal schema, or constitutional rule |
| **MINOR** | New functionality added in a backward-compatible manner |
| **PATCH** | Backward-compatible bug fixes or documentation corrections |

### 7.2 Document Versioning

Specification documents (including this one) use a parallel versioning scheme:

| Increment | When to Use |
|---|---|
| **MAJOR** | Architectural rethink, constitutional amendment, or fundamental philosophy change |
| **MINOR** | New section added, existing section substantially revised |
| **PATCH** | Typographic corrections, clarifications without substance change |

Document version must be updated on every commit that changes the document content.

### 7.3 Signal Schema Versioning

The Webhook Payload Schema is independently versioned. The schema version is carried in every signal payload as the `schema_version` field. Hermes must support the current version and one previous major version concurrently during any migration period. Migration periods must not exceed 30 days.

### 7.4 Model Versioning

Every AI model deployed to production is assigned a unique version identifier in the format `<model_name>-v<MAJOR>.<MINOR>.<PATCH>`. The active model version is logged with every signal decision. Rolling back to a previous model version must be possible within 5 minutes via configuration change (no code deployment required).

### 7.5 Changelog Requirements

Every change to the repository — code, documentation, or configuration — must have a corresponding entry in `CHANGELOG.md` following the Keep a Changelog format. The changelog entry must reference the PR or commit SHA.

---

## 8. Coding Standards

### 8.1 Python Standards

**Style**
- PEP 8 compliance is mandatory.
- Formatter: `black` (line length 100).
- Linter: `ruff` with the following rule groups enabled: `E`, `W`, `F`, `I`, `N`, `UP`, `B`, `A`, `C4`, `S` (security).
- All linting must pass with zero warnings in CI before merge.

**Type Safety**
- All function signatures must include full type annotations (PEP 484).
- Return types must be explicitly annotated; `-> None` is required where no value is returned.
- `mypy` strict mode must pass without error.
- Use of `Any` is permitted only in adapter/bridge layers interfacing with untyped third-party libraries, and must be documented with a `# type: ignore[assignment]  # reason:` comment.

**Error Handling**
- All exceptions at module boundaries must be caught, logged with full context, and either re-raised as a typed domain exception or handled explicitly.
- Bare `except:` clauses are prohibited.
- `except Exception as e:` is permitted only at the outermost service boundary with mandatory logging.
- Domain exceptions are defined in a dedicated `exceptions.py` module per service.

**Testing**
- Test framework: `pytest`.
- Minimum coverage threshold: **80%** for all non-trivial modules; **95%** for signal pipeline and risk calculation modules.
- Unit tests must not make network calls or database connections; use fixtures and mocks.
- Integration tests are clearly separated from unit tests and run in a dedicated CI stage.
- Every bug fix must include a regression test that would have caught the bug.

**Imports and Dependencies**
- Imports are sorted and grouped: standard library, third-party, internal (enforced by `ruff isort`).
- Circular imports are prohibited.
- Internal imports use absolute paths; relative imports are prohibited except within a single sub-package.

### 8.2 Naming Conventions

| Entity | Convention | Example |
|---|---|---|
| Module / Package | `snake_case` | `signal_pipeline.py` |
| Class | `PascalCase` | `SignalValidator` |
| Function / Method | `snake_case` | `validate_signal()` |
| Variable | `snake_case` | `confidence_score` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RISK_PER_TRADE` |
| Type Alias | `PascalCase` | `SignalAction` |
| Database Table | `snake_case` (plural) | `signal_events` |
| Database Column | `snake_case` | `created_at` |
| API Endpoint | `kebab-case` | `/api/v1/signal-events` |
| Environment Variable | `UPPER_SNAKE_CASE` | `WEBHOOK_SECRET` |
| Git Branch | `kebab-case` with type prefix | `feature/hermes-webhook-receiver` |

### 8.3 Documentation Requirements

- Every public function, method, and class must have a docstring.
- Docstring format: Google style.
- Docstrings must document: purpose, parameters (name, type, description), return value, and any exceptions raised.
- Complex algorithmic logic must include inline comments explaining the mathematical intent, not restating the code.
- All magic numbers must be replaced by named constants with a docstring or inline comment explaining their derivation.

### 8.4 MQL5 Standards

- All Expert Advisor code must include a header block with version, date, author, and description.
- Functions must be named descriptively in `PascalCase`.
- Error handling must use `GetLastError()` with explicit logging.
- No hard-coded lot sizes, stop-loss values, or take-profit values. All parameters must be declared as `input` variables.

---

## 9. Performance Requirements

### 9.1 Latency Requirements

| Path | Requirement | Measurement Point |
|---|---|---|
| Webhook receive → Signal validated | ≤ 100 ms (p95) | Hermes internal timer |
| Signal validated → AI scored | ≤ 500 ms (p95) | Hermes internal timer |
| AI scored → Order dispatched to MT5 | ≤ 200 ms (p95) | Hermes internal timer |
| **Total: Webhook receive → Order dispatched** | **≤ 800 ms (p95)** | End-to-end correlation trace |
| MT5 bridge → Order acknowledged by broker | ≤ 2 000 ms (p95) | MT5 bridge timer |
| Signal event → Database persisted | ≤ 200 ms (p95) | Async; must not block signal path |
| Dashboard data refresh | ≤ 5 s | Client-observed latency |

### 9.2 Throughput Requirements

| Scenario | Requirement |
|---|---|
| Peak simultaneous webhook requests | ≥ 10 requests/second sustained |
| Burst tolerance (≤ 5 seconds) | ≥ 50 requests/second |
| Database write throughput | ≥ 100 signal events/second |

*Note: XAUUSD and BTCUSD are each expected to generate at most 5–20 signals per day under normal strategy operation. The throughput requirements exist to ensure the platform handles alert storms or misconfiguration gracefully without data loss.*

### 9.3 Availability Requirements

| Component | Target Availability |
|---|---|
| Hermes Engine | 99.9% (< 44 minutes downtime/month) |
| MT5 Bridge | 99.9% during market hours |
| Database | 99.95% |
| Dashboard | 99.0% (non-critical path) |

### 9.4 Resource Constraints

- The Hermes engine must operate within 512 MB RAM under normal load.
- The MT5 bridge must not consume more than 1 CPU core at sustained load.
- Database storage growth must be projected quarterly; retention policies must archive data older than 2 years without deleting audit records.

### 9.5 Performance Testing

- A performance test suite must be established before Phase 1 production deployment.
- Latency requirements must be validated under simulated load before any production promotion.
- Performance regression tests must run in CI on a weekly schedule.

---

## 10. Non-Repainting Requirements

### 10.1 Definition

A signal **repaints** when its value changes after being plotted or calculated for a bar that is no longer the currently forming (live) bar. Repainting causes backtests to show trades that could not have been taken in real time, producing false performance metrics.

### 10.2 The Non-Repainting Standard

**All signal calculations in this platform, in all components, must be computed exclusively on confirmed, closed bars.**

This is a Constitutional Rule (Rule III) and admits no exceptions.

### 10.3 Enforcement in Pine Script

- Signal logic must always reference `bar_index[1]` or use `barstate.isconfirmed` as a guard.
- The `security()` function must use `lookahead = barmerge.lookahead_off` at all times.
- No signal entry condition may use the current (live) bar's `close`, `high`, `low`, or any derived value calculated from the live bar.
- All Pine Script strategies must include the header comment `// NON-REPAINTING: Signals on confirmed bars only` and a mechanism to validate this in automated review.

### 10.4 Enforcement in Hermes

- Any signal re-computation or enrichment performed in Hermes that references market data must use OHLCV data sourced with an explicit time filter that excludes the current incomplete bar.
- The data retrieval layer must enforce this by design: the `get_ohlcv()` function must accept a `closed_bars_only: bool = True` parameter that defaults to `True` and raises a `NonRepaintingViolationError` if called with `False` in a production environment.

### 10.5 Audit Procedure

Every signal payload must include a `bar_close_time` field containing the UTC close time of the bar that generated the signal. The Hermes validator must verify that `bar_close_time < signal_received_time`. Signals where `bar_close_time >= signal_received_time` are rejected with a `REPAINTING_SIGNAL_REJECTED` error code and logged as a critical warning.

---

## 11. Risk Hierarchy

### 11.1 Risk Authority Levels

Risk controls are organized into a three-tier hierarchy. Higher-tier controls supersede lower-tier controls unconditionally.

```
┌────────────────────────────────────────────────────────┐
│  TIER 1 — CONSTITUTIONAL LIMITS (Hardcoded at deploy)  │
│  Cannot be changed without a platform restart and      │
│  formal amendment. Override requires two-person auth.  │
├────────────────────────────────────────────────────────┤
│  TIER 2 — OPERATOR LIMITS (Runtime configuration)      │
│  Set by the operator via environment variables.        │
│  Changes take effect on service restart.               │
├────────────────────────────────────────────────────────┤
│  TIER 3 — STRATEGY LIMITS (Per-signal parameters)      │
│  Suggested by the signal payload (stop-loss,           │
│  take-profit). Subject to Tier 1 and Tier 2 override.  │
└────────────────────────────────────────────────────────┘
```

### 11.2 Tier 1 — Constitutional Risk Limits

| Parameter | Value | Notes |
|---|---|---|
| Maximum single-trade risk | 5% of account equity | Hard ceiling; Tier 2 may set lower |
| Maximum simultaneous open positions | 5 | Across all instruments |
| Maximum daily loss | 10% of account equity | Platform halts on breach |
| Maximum total drawdown | 20% of peak equity | Platform halts on breach; human intervention required to resume |
| Minimum stop-loss distance | 5 pips / 50 points | Prevents accidental near-zero stop placement |

### 11.3 Tier 2 — Operator Risk Parameters

Operator-configurable parameters (environment variables). All must be set within the Tier 1 bounds.

| Parameter | Environment Variable | Default |
|---|---|---|
| Risk per trade (% of equity) | `RISK_PER_TRADE_PCT` | `1.0` |
| Max simultaneous positions | `MAX_OPEN_POSITIONS` | `3` |
| Daily loss limit (% of equity) | `DAILY_LOSS_LIMIT_PCT` | `5.0` |
| Max drawdown halt threshold (%) | `MAX_DRAWDOWN_HALT_PCT` | `15.0` |
| Confidence score threshold (pass) | `AI_CONFIDENCE_THRESHOLD` | `0.65` |

### 11.4 Tier 3 — Signal-Level Parameters

Signal payloads may suggest stop-loss and take-profit levels. These suggestions are:
- Validated against minimum stop-loss distance (Tier 1).
- Validated against the resulting position size does not exceed the Tier 2 risk-per-trade limit.
- Overridden by Hermes if they violate any Tier 1 or Tier 2 constraint.

### 11.5 Risk Calculation Standard

Position size is calculated using the **Fixed Fractional method** adjusted for volatility:

```
Lot Size = (Account Equity × Risk Per Trade %) / (Stop Loss in Points × Point Value)
```

For volatility adjustment, the stop-loss distance is replaced by a multiple of the current ATR:

```
ATR Stop Distance = ATR(n) × ATR Multiplier
Lot Size = (Account Equity × Risk Per Trade %) / (ATR Stop Distance × Point Value)
```

Where:
- `ATR(n)` is the Average True Range over `n` periods (default n=14) of the signal timeframe.
- `ATR Multiplier` is a strategy-specific configurable parameter.
- `Point Value` is the monetary value of one point movement for the instrument.

### 11.6 Platform Halt Procedure

When the daily loss limit or maximum drawdown halt threshold is breached:

1. All new signal processing is immediately suspended.
2. All pending orders are cancelled.
3. Existing open positions are **not** automatically closed (to avoid panic liquidation).
4. An operator alert is sent within 30 seconds.
5. A human operator must explicitly resume the platform via the control interface.
6. The halt event is recorded in the audit log with full context.

---

## 12. AI Hierarchy

### 12.1 AI System Role

The AI layer within Hermes is a **signal enrichment and gating** system. It does not generate signals. It does not override Constitutional or Tier 1 risk controls. Its function is to improve the quality of signals that pass through to execution by filtering out low-confidence opportunities.

### 12.2 AI Authority

```
┌─────────────────────────────────────────────────────────────────┐
│  WHAT THE AI MAY DO                                             │
│  • Score signals with a confidence value [0.0 – 1.0]           │
│  • Classify current market regime                               │
│  • Recommend signal PASS or FILTER                              │
│  • Adjust suggested position size (downward only, within Tier 2)│
├─────────────────────────────────────────────────────────────────┤
│  WHAT THE AI MUST NOT DO                                        │
│  • Generate a signal autonomously (no AI-only signals)          │
│  • Override a risk limit at any tier                            │
│  • Modify stop-loss or take-profit beyond what Tier 2 allows    │
│  • Execute or cancel orders directly                            │
│  • Operate in production without human-reviewed evidence        │
└─────────────────────────────────────────────────────────────────┘
```

### 12.3 AI Decision Transparency

Every AI decision must be logged with:

| Field | Description |
|---|---|
| `model_version` | Version identifier of the model that produced the decision |
| `confidence_score` | Numeric confidence score [0.0 – 1.0] |
| `market_regime` | Detected market regime classification |
| `feature_snapshot` | The input feature vector used for inference (for reproducibility) |
| `decision` | `PASS` or `FILTER` |
| `decision_reason` | Human-readable summary of primary decision factors |
| `inference_latency_ms` | Time taken for inference |

### 12.4 AI Failure Handling

If the AI model fails to produce a decision (timeout, exception, or unavailable):

- **Conservative mode (default):** The signal is filtered (not passed to execution). This is the recommended production default.
- **Permissive mode (operator-configured):** The signal is passed with a default confidence score of 0.0 and flagged as `AI_BYPASS`. This mode requires explicit operator enablement and is not permitted to be the default.

The AI bypass mode must be logged as a high-severity event in the audit log.

### 12.5 AI Model Governance

| Requirement | Standard |
|---|---|
| All models must pass Evidence Gates 1–3 before production | Section 6.2 |
| Models must be retrained when drift is detected | Drift detection procedure TBD in Phase 5 |
| A model rollback must be possible within 5 minutes | Via `AI_MODEL_VERSION` environment variable change |
| Model training data must be versioned and archived | In the `data/` directory with a manifest |
| Shadow mode deployment is required for new model versions | New model runs in parallel for 2 weeks before promotion |

---

## 13. Module Responsibilities

### 13.1 Responsibility Matrix

Each module has a single-sentence primary responsibility. Any function or feature that does not clearly fall within a module's primary responsibility must be assigned to the correct module before implementation.

| Module | Primary Responsibility | May NOT Do |
|---|---|---|
| **TradingView (Pine Script)** | Detect and emit trading signals based on confirmed price action and indicator logic | Execute orders; access external APIs; perform risk sizing |
| **Hermes Receiver** | Accept, authenticate, and deserialize incoming webhook payloads | Execute business logic; persist data directly; communicate with MT5 |
| **Hermes Signal Pipeline** | Validate, normalize, and enrich signal payloads into domain objects | Communicate with external systems; execute orders |
| **Hermes AI Layer** | Score signal confidence and classify market regime | Generate signals; override risk controls; communicate with MT5 directly |
| **Hermes Dispatcher** | Route approved signals to the appropriate downstream consumer (MT5 bridge, database) | Apply risk calculations; score signals |
| **MT5 Bridge** | Translate approved signal commands into MT5 orders and manage position lifecycle | Apply AI scoring; validate signal schema; own signal business logic |
| **Database Layer** | Provide durable, append-only storage for all signal events, trade records, and audit logs | Apply business logic; make trading decisions |
| **Dashboard** | Display real-time and historical platform state to human operators | Send signals; modify positions; change configuration |
| **Configuration System** | Supply runtime parameters to all services from a single source of truth | Apply business logic; store state |

### 13.2 TradingView Module

**Responsibilities:**
- Implement signal detection logic in Pine Script v5 on confirmed, closed bars.
- Configure TradingView alerts to fire HTTPS webhooks to the Hermes Receiver endpoint.
- Produce a well-formed payload conforming to the Webhook Payload Schema v1.
- Maintain separate strategies for XAUUSD and BTCUSD.
- Include strategy metadata (version, strategy name) in every signal payload.

**Constraints:**
- Must satisfy all non-repainting requirements (Section 10.3).
- Must not use `lookahead` in `security()` calls.
- Must version-tag every Pine Script strategy change.

### 13.3 Hermes Engine Module

**Responsibilities:**
- Receive and authenticate webhook requests.
- Validate payload schema and business rules.
- Normalize signals into internal domain objects.
- Invoke the AI layer for enrichment.
- Apply Tier 2 risk pre-checks.
- Dispatch approved signals.
- Persist all events to the database asynchronously.
- Expose health and status API endpoints.
- Implement the operator control interface (halt/resume/module disable).

**Constraints:**
- Must be stateless per request.
- Must meet all latency requirements (Section 9.1).
- Must implement all observability standards (Section 4.5).

### 13.4 MT5 Bridge Module

**Responsibilities:**
- Maintain a persistent, authenticated connection to the MT5 terminal.
- Translate approved signal commands into MT5 order requests.
- Apply final Tier 1 risk pre-checks at the execution boundary.
- Manage open position lifecycle (modify, close).
- Report execution outcomes (fills, rejections, partial fills) back to Hermes.
- Persist trade records to the database.

**Constraints:**
- Must perform final Tier 1 risk checks independently of Hermes (defense in depth).
- Must handle MT5 connection failures gracefully (fail closed).
- Must never open a position without a valid, traceable `signal_id`.

### 13.5 Database Module

**Responsibilities:**
- Maintain the canonical data schema.
- Provide a migration framework for schema evolution.
- Own all index and performance optimization for query patterns.
- Enforce append-only constraints on the audit log table.
- Provide seed data for development and testing environments.

**Constraints:**
- The `audit_log` table must have a database-level trigger that prevents `UPDATE` and `DELETE` operations.
- Schema migrations must be reversible (down migrations required for all changes).
- No application logic in stored procedures or triggers beyond integrity enforcement.

### 13.6 Dashboard Module

**Responsibilities:**
- Display the live signal feed and active positions.
- Display performance metrics and PnL history.
- Provide the operator control interface (halt, resume, module control).
- Provide authenticated access (no anonymous access permitted).

**Constraints:**
- Read-only access to the database (dedicated read-only database user).
- Control actions are proxied through the Hermes control API, not executed directly against the database.

---

## 14. Amendment Process

### 14.1 Who May Propose Amendments

Any contributor may propose an amendment to this specification by opening a GitHub Issue using the `[ITS Amendment]` label. The issue must state:
- The section(s) affected.
- The proposed change.
- The rationale.
- The impact on dependent specifications and existing implementations.

### 14.2 Amendment Approval

| Change Type | Approval Required |
|---|---|
| Constitutional Rule change (Section 3) | Repository owner + minimum one senior contributor review |
| Risk Hierarchy change (Section 11) | Repository owner review |
| Technical Standards change (Section 4, 8) | Minimum one senior contributor review |
| Performance Requirement change (Section 9) | Minimum one senior contributor review |
| Other sections | Standard PR review |

### 14.3 Version Increment on Amendment

Every approved amendment that changes the substance of this document must result in a version increment following the rules in Section 7.2, a CHANGELOG entry, and an updated `Last Updated` date on this document.

---

*End of Institutional Trading Specification — ITS v1.0*

*This document is the constitutional foundation of mdtsignal_fx. All contributors are bound by its rules.*
