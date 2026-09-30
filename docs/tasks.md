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

## Deploy (CCE + exposure) — DONE 2026-09-30

8. ✅ nginx pod spec serving `index.html` on port 8080 (unprivileged image), HTML from ConfigMap `claro-cenam-html`.
9. ✅ Deployed to CCE cluster **`cce-dashboard`** (v1.35), region **la-north-2**, namespace **`claro-cenam`**.
10. ✅ Exposed via dedicated **ELB** (autocreate, public IP **149.232.142.100**), port 80.
11. ✅ Public URL verified: **http://149.232.142.100/** — HTTP 200, contains "Claro CENAM"; the wrong clock is live.
12. ✅ GitHub issue created: **https://github.com/jarceg884/claro-cenam/issues/1** — "Fix: Clock shows wrong timezone (UTC+9 instead of Central America UTC-6)".

Deploy manifests: `k8s/deployment-elb.yaml` (namespace, deployment, service with ELB autocreate).

## Live-demo runbook (during the session)

> Prereq: page deployed and reachable at **http://149.232.142.100/**; issue #1 open. Do all steps before the audience joins.
> Cluster: **cce-dashboard** (v1.35). Kubeconfig: `C:/Users/j84403562/Downloads/cce-dashboard-kubeconfig.yaml` (context `external`). kubectl: `$TMPDIR/kubectl.exe`.

1. **Show the bug** — open **http://149.232.142.100/** in the browser; point out the clock is 15 hours ahead while labeled "Local Time — CENAM".
2. **Open the issue** — show https://github.com/jarceg884/claro-cenam/issues/1; read the description aloud.
3. **Fix one line** — in `index.html`, change `const TIMEZONE = 'Asia/Tokyo';` → `const TIMEZONE = 'America/Guatemala';`. Show the one-line diff.
4. **Commit & push** — `git commit -m "fix: use America/Guatemala timezone for CENAM clock (fixes #1)"` then `git push`.
5. **Redeploy the pod** — update the ConfigMap and roll the pod:
   ```
   export KUBECONFIG="C:/Users/j84403562/Downloads/cce-dashboard-kubeconfig.yaml"
   kubectl create configmap claro-cenam-html --from-file=index.html=index.html -n claro-cenam --dry-run=client -o yaml | kubectl apply -f -
   kubectl rollout restart deployment/claro-cenam -n claro-cenam
   kubectl rollout status deployment/claro-cenam -n claro-cenam
   ```
6. **Verify** — hard-refresh the page; the clock now shows correct Guatemala time (UTC−6) matching local time.
7. **Close the loop** — the push with `(fixes #1)` auto-closes the issue; show it closed on GitHub.

### Rollback / contingency

- If the redeploy stalls: `kubectl rollout undo deployment/claro-cenam -n claro-cenam` and present the diff + local preview instead.
- If the ELB is unreachable: present via `kubectl port-forward -n claro-cenam svc/claro-cenam 8080:80` and a local browser.
