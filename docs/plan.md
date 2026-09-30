# Claro CENAM — Plan

## 1. Architecture

```
Browser ──► ELB (public IP, la-north-2)
              │
              ▼
       CCE cluster "cce-dashboard" (v1.35)
              │
              ▼
       Service (type: LoadBalancer)
              │
              ▼
       Pod: nginx (serves static page :80)
              │
              ▼
       index.html — "Claro CENAM"
         Claro branding + JS real-time clock
         (bug: clock renders Asia/Tokyo, labeled CENAM)
```

- **Frontend:** one static `index.html` (self-contained or with sibling CSS/JS). Claro red #DA291C, Claro wordmark, real-time clock updated via `setInterval` every second. No build step.
- **Container:** stock `nginx` image with the page mounted/copied in; served on port 80.
- **Cluster:** Huawei Cloud CCE `cce-dashboard` (v1.35), region **la-north-2**.
- **Exposure:** Kubernetes `Service` of type `LoadBalancer` → provisions a Huawei ELB with a public IP; port 80.

## 2. Key decisions

| Decision | Choice | Why |
|----------|--------|-----|
| Page type | Static HTML, no framework | Demo-scale; the "fix" must be a one-line change visible in a diff |
| Clock implementation | Client-side JS, `Intl`/`toLocaleTimeString` with a hard-coded timezone string | The bug and fix are a single timezone string (`Asia/Tokyo` → `America/Guatemala`) |
| Web server | nginx in a pod | Simplest reliable static server; standard CCE pattern |
| Exposure | LoadBalancer Service (ELB) | Fastest path to a public URL on CCE; no ingress controller needed |
| Bug placement | Timezone constant in the page's JS | Fix is one line, no rebuild logic — ideal for a live demo |
| Correct timezone | `America/Guatemala` (UTC−6) | CENAM = Central America; engineer's location |
| Repo | Public `jarceg884/claro-cenam` | Audience must view code and issue live |

## 3. Timezone bug design

- **Before (shipped bug):** clock uses `Asia/Tokyo` (UTC+9); label says "CENAM Local Time". Clock is **15 hours ahead** of Guatemala.
- **After (fix):** clock uses `America/Guatemala` (UTC−6); label unchanged and now accurate.
- The fix must be exactly **one line** so the live diff is trivial to present.

## 4. Risks / notes

- **ELB IP provisioning time:** a public ELB can take 1–3 minutes to get its IP on first Service creation — create it before the demo, not during.
- **Redeploy speed:** use `kubectl rollout restart` or a rolling image/pod update so the fix goes live in seconds; avoid delete-and-recreate.
- **Cache:** ensure nginx serves `index.html` without aggressive caching (or instruct demo to hard-refresh) so the fixed clock appears immediately.
- **Scope guard:** this is a tiny static page — no CI/CD pipeline, no TLS, no custom domain for the demo.
