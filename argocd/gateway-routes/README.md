# Gateway HTTPRoutes (main-gateway in kube-system)

These HTTPRoutes attach to `main-gateway` (kube-system) so traffic uses the same TLS cert (aioc-services-tls) as your app.

- **promstack-grafana.yaml** — grafana.aioc-services.com → promstack-grafana:3000
- **promstack-prometheus.yaml** — prometheus.aioc-services.com → promstack-kube-prometheus:9090
- **promstack-alertmanager.yaml** — alertmanager.aioc-services.com → promstack-kube-prometheus-alertmanager:9093

Apply: `kubectl apply -f argocd/gateway-routes/` or use the Argo CD Application `gateway-routes-promstack.yaml`.
