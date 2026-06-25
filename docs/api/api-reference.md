# API Reference

## Title
mdtsignal_fx — API Reference

## Purpose
This document is the authoritative reference for all HTTP API endpoints exposed by the mdtsignal_fx platform, including the Hermes webhook receiver and any monitoring or management APIs.

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
2. [Authentication](#authentication)
3. [Base URL](#base-url)
4. [Endpoints](#endpoints)
   - [Webhook](#webhook)
   - [Health](#health)
   - [Signals](#signals)
   - [Positions](#positions)
5. [Error Codes](#error-codes)
6. [Rate Limiting](#rate-limiting)

---

## Overview

The mdtsignal_fx API is a RESTful HTTP API used by TradingView to deliver signal webhooks and by the dashboard to retrieve live platform state.

_[OpenAPI/Swagger specification to be generated during Phase 1.]_

---

## Authentication

_[Authentication mechanism (API key / JWT) to be defined during Phase 1.]_

---

## Base URL

```
https://<host>/api/v1
```

---

## Endpoints

### Webhook

```
POST /webhook/signal
```
Receives a signal payload from TradingView.

_[Full request/response schema to be defined.]_

---

### Health

```
GET /health
```
Returns platform health status.

---

### Signals

```
GET /signals
GET /signals/{id}
```
Retrieve processed signals.

_[Query parameters and response schema to be defined.]_

---

### Positions

```
GET /positions
```
Retrieve current open positions.

_[Response schema to be defined.]_

---

## Error Codes

| Code | Meaning |
|---|---|
| 400 | Bad Request — invalid payload |
| 401 | Unauthorized — missing or invalid credentials |
| 422 | Unprocessable Entity — validation failure |
| 500 | Internal Server Error |

---

## Rate Limiting

_[Rate limiting policy to be defined.]_
