# ITS Overview

## Title
Integrated Trading System (ITS) — Overview

## Purpose
This document provides a high-level overview of the Integrated Trading System architecture for mdtsignal_fx. It describes how all platform components interact to form a cohesive, end-to-end trading pipeline from signal generation to order execution. For the full constitutional specification, see [ITS v1.0](ITS-v1.0.md).

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

1. [System Overview](#system-overview)
2. [Component Map](#component-map)
3. [Data Flow](#data-flow)
4. [Integration Points](#integration-points)
5. [Failure Modes and Resilience](#failure-modes-and-resilience)
6. [Related Specifications](#related-specifications)

---

## System Overview

The Integrated Trading System (ITS) is the architectural backbone of mdtsignal_fx. It defines how market signals originating from TradingView Pine Script strategies are received, validated, enriched by the Hermes AI engine, and ultimately executed via the MetaTrader 5 bridge.

_[Content to be elaborated during Phase 1 development.]_

---

## Component Map

| Component | Role |
|---|---|
| TradingView | Signal source (Pine Script + webhook) |
| Hermes | Signal engine and AI decision layer |
| MT5 Bridge | Order execution gateway |
| Dashboard | Real-time monitoring |
| Database | Persistent signal, trade, and audit storage |

---

## Data Flow

```
TradingView Alert
      │
      ▼ (HTTPS Webhook)
Hermes Receiver
      │
      ▼ (Validation & Enrichment)
AI Decision Layer
      │
      ▼ (Approved Signal)
MT5 Bridge
      │
      ▼ (Order Execution)
Database (Audit Log)
      │
      ▼
Dashboard (Live Feed)
```

---

## Integration Points

_[To be defined during Phase 1 and Phase 2 development.]_

---

## Failure Modes and Resilience

_[To be defined during Phase 6 production hardening.]_

---

## Related Specifications

- [Signal Specification](signal-specification.md)
- [Risk Management Specification](risk-management.md)
- [Execution Specification](execution-specification.md)
