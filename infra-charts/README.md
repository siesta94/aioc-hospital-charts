# Infra charts

Cluster infrastructure Helm charts (monitoring, etc.). Deployed by Argo CD; apps live under `app-charts/`.

## promstack

Wrapper around [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack): Prometheus, Grafana, Alertmanager, node-exporter, kube-state-metrics.

- **Path:** `infra-charts/promstack`
- **Namespace:** `monitoring`
- **Argo CD:** `argocd/applications/promstack.yaml` — apply with `kubectl apply -f argocd/applications/promstack.yaml`

**Grafana admin:** Password is read from a Secret (not stored in repo). Create it before or after deploy:

```bash
kubectl create secret generic grafana-admin -n monitoring \
  --from-literal=admin-user=admin \
  --from-literal=admin-password='YOUR_PASSWORD'
```

**Domains (Grafana, Prometheus, Alertmanager):** Promstack is exposed via the **same Gateway** as your app (`main-gateway` in kube-system), so it uses the **same wildcard cert** (aioc-services-tls in kube-system). No extra Certificate needed.

- **HTTPRoutes** are in `argocd/gateway-routes/` (grafana, prometheus, alertmanager). They attach to main-gateway and route to the Promstack services in monitoring.
- **Apply:** `kubectl apply -f argocd/applications/gateway-routes-promstack.yaml` so Argo CD syncs the routes, or apply once: `kubectl apply -f argocd/gateway-routes/`.
- **Hostnames:** grafana.aioc-services.com, prometheus.aioc-services.com, alertmanager.aioc-services.com (edit the HTTPRoute YAMLs to change).
- Point DNS for those hostnames to the same external IP as your app (the Gateway's LB).
