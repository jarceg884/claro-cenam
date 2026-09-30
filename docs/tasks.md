# Claro CENAM — Tasks

## Dev (build the page)

1. Create `index.html` titled **"Claro CENAM"** with Claro branding (red #DA291C, Claro wordmark/logo).
2. Add a real-time clock that updates every second (client-side JS).
3. **Ship the bug:** hard-code the clock timezone to `Asia/Tokyo` while the label reads "CENAM Local Time". Do NOT fix this — it is intentional.
4. Commit and push to `github.com/jarceg884/claro-cenam` (default branch).

## QA (verify the bug is present and presentable)

5. Open the page locally; confirm the clock ticks every second and shows Tokyo time (UTC+9, i.e. 15 hours ahead of Guatemala) under a "CENAM Local Time" label.
6. Confirm the fix will be a **one-line** timezone change (`Asia/Tokyo` → `America/Guatemala`) — no other edits needed.
7. Confirm Claro branding renders correctly (red, logo, layout).

## Deploy (CCE + exposure)

8. Build/prepare the nginx pod spec serving `index.html` on port 80.
9. Deploy the pod to CCE cluster **`cce-dashboard`** (v1.35), region **la-north-2**.
10. Create a `Service` of type `LoadBalancer` (ELB) exposing port 80; wait for the public IP.
11. Verify the public URL: `curl -s http://<ELB-IP>/` returns 200 and contains "Claro CENAM"; the wrong clock is visible.
12. Create GitHub issue **"fix the clock"** in `jarceg884/claro-cenam` describing: clock shows Asia/Tokyo (UTC+9) labeled as CENAM local time; expected fix = `America/Guatemala` (UTC−6); one-line change + redeploy.

## Live-demo runbook (during the session)

> Prereq: page deployed and reachable at the ELB public URL; issue "fix the clock" open. Do all steps before the audience joins.

1. **Show the bug** — open `http://<ELB-IP>/` in the browser; point out the clock is 15 hours ahead while labeled "CENAM Local Time".
2. **Open the issue** — show GitHub issue "fix the clock"; read the description aloud.
3. **Fix one line** — in the page source, change the timezone string `Asia/Tokyo` → `America/Guatemala`. Show the one-line diff.
4. **Commit & push** — `git commit -m "fix: use America/Guatemala timezone for CENAM clock (fixes #N)"` then `git push`.
5. **Redeploy the pod** — rolling update on CCE (e.g. `kubectl rollout restart deployment/<name>` or re-apply the pod spec with the updated page); wait for rollout to complete.
6. **Verify** — hard-refresh the page; the clock now shows correct Guatemala time (UTC−6) matching local time.
7. **Close the loop** — comment on / close the GitHub issue referencing the fix commit.

### Rollback / contingency

- If the redeploy stalls: `kubectl rollout undo deployment/<name>` and present the diff + local preview instead.
- If the ELB is unreachable: present via `kubectl port-forward` and a local browser.
