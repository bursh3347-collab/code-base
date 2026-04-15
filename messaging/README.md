# 📨 Messaging & Gateway Architecture

> Extracted messaging patterns, event-driven architectures, and multi-platform gateway designs from high-star projects.

## 📦 What's Inside

| Pattern | Source | Type | Best For |
|---------|--------|------|----------|
| Multi-Platform Gateway | Hermes Agent (86k⭐) | Gateway architecture | Multi-channel bots/agents |
| Event-Driven (Pub/Sub) | Various | Event bus | Decoupled microservices |
| WebSocket | Various | Real-time | Live updates, chat |
| Webhooks | Various | Push notifications | Third-party integrations |

## 🎯 Selection Guide

```
Need real-time?
├── Yes
│   ├── Browser client? → WebSocket / SSE
│   └── Multi-platform bot? → Gateway pattern (from Hermes)
└── No
    ├── Service-to-service? → Event bus / message queue
    └── Third-party integration? → Webhooks
```

## 📁 Directory Structure

```
messaging/
├── README.md              ← You are here
├── gateway/               ← Multi-platform gateway patterns (from Hermes)
├── websocket/             ← WebSocket patterns
├── webhooks/              ← Webhook receiver/sender patterns
└── event-bus/             ← Event-driven architecture patterns
```

## 🏷️ Origin Note

The multi-platform gateway pattern was identified during analysis of [NousResearch/hermes-agent](https://github.com/nousresearch/hermes-agent) (86k⭐, MIT). It supports 6 terminal backends (local/Docker/SSH/Daytona/Singularity/Modal) — a cross-domain reusable architecture.

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
