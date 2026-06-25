# Webhook Payload Schema

## Title
API — Webhook Payload Schema

## Purpose
This document defines the exact JSON payload schema that TradingView must send to the Hermes webhook endpoint when firing a signal alert. It is the contract between TradingView and the Hermes signal engine.

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

1. [Overview](#overview)
2. [Payload Schema](#payload-schema)
3. [Field Descriptions](#field-descriptions)
4. [Example Payloads](#example-payloads)
5. [Validation Rules](#validation-rules)
6. [TradingView Alert Configuration](#tradingview-alert-configuration)

---

## Overview

TradingView delivers signal data via HTTPS POST to the Hermes webhook receiver. The payload must conform to the JSON schema defined in this document.

---

## Payload Schema

_[JSON Schema (Draft 7) to be defined during Phase 1/2.]_

---

## Field Descriptions

| Field | Type | Required | Description |
|---|---|---|---|
| `symbol` | string | ✅ | Instrument symbol (e.g. `XAUUSD`) |
| `action` | string | ✅ | Signal action (`ENTRY_LONG`, `ENTRY_SHORT`, `CLOSE_LONG`, `CLOSE_SHORT`) |
| `price` | number | ✅ | Signal trigger price |
| `timestamp` | string (ISO 8601) | ✅ | Signal generation time in UTC |
| `strategy` | string | ✅ | Originating strategy identifier |
| `timeframe` | string | ✅ | Chart timeframe (e.g. `1H`, `4H`) |
| `stop_loss` | number | ❌ | Suggested stop loss price |
| `take_profit` | number | ❌ | Suggested take profit price |

---

## Example Payloads

_[Example BUY and SELL payloads to be added during Phase 2.]_

---

## Validation Rules

_[Business validation rules (e.g. symbol whitelist, price sanity checks) to be defined.]_

---

## TradingView Alert Configuration

_[Step-by-step guide to configuring TradingView webhook alerts to be added during Phase 2.]_
