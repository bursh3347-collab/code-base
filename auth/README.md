# 🔐 Authentication Patterns

> Extracted authentication implementations from high-star open-source projects.

## 📦 What's Inside

| Approach | Source Project | Complexity | Best For |
|----------|---------------|------------|----------|
| JWT | *TBD* | Low | API services, stateless auth |
| OAuth | *TBD* | Medium | Social login (Google, GitHub) |
| Magic Link | *TBD* | Low | Passwordless login, email-first |
| Session-based | *TBD* | Medium | Traditional web apps |

## 🎯 Selection Guide

```
Do you need social login?
├── Yes → OAuth (+ JWT for API layer)
└── No
    ├── API-only service? → JWT
    ├── Email-first product? → Magic Link
    └── Traditional web app? → Session-based
```

### ⭐ Recommended Default: JWT

**Why**: Simplest to implement, most flexible, compatible with most SaaS templates. Works great with Next.js + Supabase + Stripe stack.

## 📁 Directory Structure

```
auth/
├── README.md          ← You are here
├── jwt/               ← JWT authentication patterns
├── oauth/             ← OAuth integration patterns
├── magic-link/        ← Magic Link login patterns
└── session/           ← Session-based auth patterns
```

## 🔗 Cross-References

- Database patterns for user tables → [../database/](../database/)
- API middleware for auth → [../api/](../api/)
- Error handling for auth failures → [../error-handling/](../error-handling/)

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
