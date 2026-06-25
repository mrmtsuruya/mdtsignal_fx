# Hermes Service Overview

## Title
Hermes — Service Overview

## Purpose
This document provides the top-level service documentation for Hermes, the core signal processing engine of mdtsignal_fx. It describes the service's responsibilities, boundaries, configuration, and operational requirements.

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

1. [Service Description](#service-description)
2. [Responsibilities](#responsibilities)
3. [Out of Scope](#out-of-scope)
4. [Dependencies](#dependencies)
5. [Configuration](#configuration)
6. [Deployment](#deployment)
7. [Monitoring and Alerting](#monitoring-and-alerting)
8. [Related Documents](#related-documents)

---

## Service Description

Hermes is the central Python service of the mdtsignal_fx platform. It acts as the intelligence layer between external signal sources (TradingView) and the execution layer (MetaTrader 5).

---

## Responsibilities

- Receive and authenticate incoming signal webhooks
- Validate and normalize signal payloads
- Apply AI enrichment (confidence scoring, market regime detection)
- Dispatch approved signals to the MT5 bridge
- Persist all signal events to the database
- Expose a health and status API

---

## Out of Scope

- Direct broker order management (handled by MT5 bridge)
- Front-end rendering (handled by Dashboard)
- Pine Script strategy logic (handled in TradingView)

---

## Dependencies

| Dependency | Purpose |
|---|---|
| Database | Signal and audit persistence |
| MT5 Bridge | Trade execution |
| AI Model Registry | Confidence scoring inference |

---

## Configuration

_[Environment variable reference to be defined during Phase 1.]_

---

## Deployment

_[Containerization (Docker) and orchestration strategy to be defined.]_

---

## Monitoring and Alerting

_[Key metrics, health endpoints, and alerting thresholds to be defined.]_

---

## Related Documents

- [Signal Engine Architecture](../engines/signal-engine-architecture.md)
- [API Reference](../api/api-reference.md)
- [Webhook Payload Schema](../api/webhook-payload-schema.md)
