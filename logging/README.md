# 📝 Logging Patterns

> Extracted logging setups, structured logging patterns, and log management from high-star projects.

## 📦 What's Inside

| Approach | Type | Best For |
|----------|------|----------|
| Structured JSON logs | Production | Machine-parseable, searchable |
| Pino | Logger library | Fast, low-overhead Node.js logging |
| Console (dev) | Development | Human-readable during development |

## 🎯 Key Principles

1. **Structured over string** — JSON logs are searchable and parseable
2. **Levels matter** — DEBUG/INFO/WARN/ERROR, configure per environment
3. **Context enrichment** — Include request ID, user ID, operation name
4. **Don't log secrets** — Sanitize PII and credentials

### ⭐ Recommended Default: Pino

**Why**: Fastest Node.js logger, structured JSON output, great for serverless.

## 📁 Directory Structure

```
logging/
├── README.md          ← You are here
├── pino/              ← Pino setup + configuration
├── structured/        ← Structured logging patterns
└── sanitization/      ← Log sanitization (PII, secrets)
```

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
