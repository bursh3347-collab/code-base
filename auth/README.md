# 🔐 Auth — Authentication Modules

> Battle-tested authentication patterns extracted from high-star open-source projects. All code is production-ready TypeScript for Next.js 14+ App Router.

## Quick Selection Guide

| Pattern | Best For | Complexity | Edge Compatible | Source |
|---------|----------|------------|-----------------|--------|
| **[JWT Middleware](./jwt-middleware.md)** | API services, microservices | Low | ✅ Yes | open-saas 14k⭐ |
| **[OAuth Social Login](./oauth-social-login.md)** | SaaS with Google/GitHub login | Medium | ⚠️ Partial | NextAuth.js 25k⭐ |
| **[Magic Link](./magic-link.md)** | Passwordless, consumer apps | Low | ✅ Yes | Supabase Auth |

## ⭐ Recommended

**If you're not sure which to pick, start with Magic Link (Supabase).**

Reasons:
- Simplest setup — zero password logic, zero OAuth credential management
- Supabase handles email delivery, session management, and token refresh
- Pairs perfectly with Supabase database (RLS policies use `auth.uid()`)
- Easy to add OAuth providers later via Supabase dashboard

## When to Use What

```
┌─ Building an API / microservice? → JWT Middleware
│
├─ Need Google/GitHub login? → OAuth Social Login
│  └─ Already using Supabase? → Use Supabase OAuth instead of NextAuth
│
└─ Want passwordless / simplest path? → Magic Link
```

## Combining Patterns

These patterns are **not mutually exclusive**:

- **Magic Link + JWT**: Use Supabase magic link for login, then issue your own JWT for API access
- **OAuth + JWT**: Use NextAuth for social login, issue JWT for stateless API routes
- **All three**: Supabase Auth supports magic link + OAuth + password in a single auth provider

## Stack Compatibility

| Component | JWT | OAuth | Magic Link |
|-----------|-----|-------|------------|
| Next.js 14+ App Router | ✅ | ✅ | ✅ |
| Vercel Edge | ✅ | ⚠️ | ✅ |
| Supabase | ✅ | ✅ | ✅ (native) |
| Stripe Integration | Manual | Built-in | Manual |
| Drizzle ORM | ✅ | ✅ (adapter) | N/A (Supabase manages) |

## Security Checklist

- [ ] JWT secret is ≥ 32 characters
- [ ] Cookies are `httpOnly`, `secure`, `sameSite: 'lax'`
- [ ] OAuth redirect URIs are explicitly whitelisted
- [ ] PKCE flow is used (not implicit grant)
- [ ] Rate limiting on auth endpoints
- [ ] Session tokens are rotated on privilege escalation
