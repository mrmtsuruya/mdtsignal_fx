# ROADMAP

## Title
mdtsignal_fx — Product Roadmap

## Purpose
This document outlines the planned milestones, features, and delivery timeline for the mdtsignal_fx platform. It serves as the single source of truth for project direction and prioritization.

## Status
Active

## Version
0.1.0

## Last Updated
2026-06-25

## Author
mrmtsuruya

---

## Table of Contents

1. [Overview](#overview)
2. [Phase 0 — Repository Foundation](#phase-0--repository-foundation)
3. [Phase 1 — Hermes Core Engine](#phase-1--hermes-core-engine)
4. [Phase 2 — TradingView Integration](#phase-2--tradingview-integration)
5. [Phase 3 — MT5 Bridge](#phase-3--mt5-bridge)
6. [Phase 4 — Dashboard](#phase-4--dashboard)
7. [Phase 5 — AI Enhancement Layer](#phase-5--ai-enhancement-layer)
8. [Phase 6 — Production Hardening](#phase-6--production-hardening)

---

## Overview

The roadmap is organized into sequential phases. Each phase delivers a production-ready, independently deployable capability. Phases may overlap once earlier phases reach stability.

---

## Phase 0 — Repository Foundation
**Target:** Q3 2026 | **Status:** ✅ Complete

- [x] Professional repository structure
- [x] Root documentation (README, ROADMAP, CHANGELOG, CONTRIBUTING, LICENSE)
- [x] Architecture and specification documents
- [x] CI/CD pipeline scaffolding
- [x] Issue and PR templates

---

## Phase 1 — Hermes Core Engine
**Target:** Q3 2026 | **Status:** 🔲 Planned

- [ ] Project scaffolding (Python package, dependency management)
- [ ] Signal data models and schemas
- [ ] Webhook receiver (TradingView → Hermes)
- [ ] Signal validation and normalization pipeline
- [ ] Configuration management (environment-based)
- [ ] Structured logging and error handling
- [ ] Unit tests with ≥80% coverage

---

## Phase 2 — TradingView Integration
**Target:** Q3 2026 | **Status:** 🔲 Planned

- [ ] Pine Script signal strategy (XAUUSD)
- [ ] Pine Script signal strategy (BTCUSD)
- [ ] Alert webhook configuration
- [ ] Payload schema documentation
- [ ] Integration tests with Hermes webhook receiver

---

## Phase 3 — MT5 Bridge
**Target:** Q4 2026 | **Status:** 🔲 Planned

- [ ] MQL5 Expert Advisor scaffold
- [ ] Python MT5 connector (MetaTrader5 library)
- [ ] Signal-to-order translation layer
- [ ] Order management (open, modify, close)
- [ ] Position sizing and risk controls
- [ ] Integration tests (paper trading)

---

## Phase 4 — Dashboard
**Target:** Q4 2026 | **Status:** 🔲 Planned

- [ ] Technology selection (framework TBD)
- [ ] Real-time signal feed display
- [ ] Active positions monitor
- [ ] PnL and performance metrics
- [ ] Alert/notification system
- [ ] Authentication and access control

---

## Phase 5 — AI Enhancement Layer
**Target:** Q1 2027 | **Status:** 🔲 Planned

- [ ] Market regime classification model
- [ ] Signal confidence scoring
- [ ] Dynamic risk parameter adjustment
- [ ] Backtesting framework integration
- [ ] Model training pipeline

---

## Phase 6 — Production Hardening
**Target:** Q1 2027 | **Status:** 🔲 Planned

- [ ] Full observability (metrics, tracing, logging)
- [ ] High availability and failover
- [ ] Security audit and penetration testing
- [ ] Disaster recovery runbooks
- [ ] Load and stress testing
- [ ] Compliance and audit logging
