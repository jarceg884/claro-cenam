# Claro CENAM — Specification

## 1. Overview

**Claro CENAM** is a small demo web page for a live software-factory session. It is a single static HTML page with Claro branding and a real-time clock. The clock intentionally displays the **wrong timezone** (Asia/Tokyo, UTC+9) while being labeled as CENAM (Central America) local time. During the live demo, a developer fixes the timezone bug following a GitHub issue and redeploys the page to a Huawei Cloud CCE Kubernetes cluster.

- **Repo:** github.com/jarceg884/claro-cenam (public)
- **Owner:** Jose David Arce González (jarceg884)
- **UI language:** English

## 2. Requirements

### R1 — Page
- Single static HTML page (plus optional CSS/JS inline or sibling files) named/titled **"Claro CENAM"**.
- Claro branding: Claro red (#DA291C), Claro logo/wordmark, clean demo-scale layout.

### R2 — Real-time clock
- A clock that updates every second, rendered client-side.
- **Intentional bug (required):** the clock renders time in **Asia/Tokyo (UTC+9)** but the label reads **"CENAM Local Time"** (or similar). The mismatch between label and actual timezone is the bug to be fixed live.

### R3 — Source control
- All code pushed to `github.com/jarceg884/claro-cenam` on the default branch.

### R4 — Deployment
- The page is served by an **nginx pod** on Huawei Cloud CCE cluster **`cce-dashboard`** (v1.35), region **la-north-2**.
- The pod is exposed to the internet via a **LoadBalancer (ELB)** Service with a public IP.

### R5 — GitHub issue
- A GitHub issue titled **"fix the clock"** exists in the repo, describing the timezone bug and the expected fix (change to **America/Guatemala, UTC-6**).

## 3. Acceptance criteria

| # | Criterion | Verification |
|---|-----------|--------------|
| AC1 | Page titled "Claro CENAM" loads from the public ELB URL | `curl -s http://<ELB-IP>/` returns 200 and contains "Claro CENAM" |
| AC2 | Claro branding visible (red #DA291C, logo/wordmark) | Visual check in browser |
| AC3 | Clock updates every second | Observe seconds incrementing in browser |
| AC4 | **Bug present pre-demo:** clock shows Asia/Tokyo time (UTC+9) while labeled as CENAM local time | Compare displayed time vs. known Guatemala time; offset is +15 hours |
| AC5 | Repo contains the page source on the default branch | `git ls-remote` / GitHub UI |
| AC6 | CCE pod running nginx serving the page | `kubectl get pods -o wide` shows Running; ELB IP reachable |
| AC7 | GitHub issue "fix the clock" open in repo | `gh issue list` / GitHub UI |
| AC8 | **Post-demo:** after the fix commit and redeploy, clock shows America/Guatemala time (UTC−6) and label matches | Re-check AC4; offset is now 0 vs. local Guatemala time |

## 4. Demo flow (live session)

1. Presenter opens the public URL — audience sees the Claro CENAM page and the wrong clock (Tokyo time labeled as CENAM time).
2. Presenter opens GitHub issue **"fix the clock"** and walks through the bug description.
3. A developer fixes **one line** (timezone `Asia/Tokyo` → `America/Guatemala`), commits, pushes.
4. The pod is redeployed (rolling update).
5. Presenter refreshes the page — clock now shows correct CENAM (Guatemala, UTC−6) time.
6. Issue is closed with a reference to the fix commit.

## 5. Out of scope

- No backend, database, authentication, or CI/CD automation beyond the manual demo steps.
- No multi-region, scaling, or monitoring configuration.
- No i18n beyond English UI labels.
