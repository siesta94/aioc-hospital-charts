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
