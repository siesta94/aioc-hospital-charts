# AIOC Hospital app charts

Application Helm charts live here. Each service chart uses **aioc-hospital-base** as a dependency. The base chart provides Deployment, Service, ServiceAccount, optional HTTPRoute (Gateway API), and supports `env` / `envFrom`.

## Layout

- **app-charts/aioc-hospital-base/** – base chart with shared templates
- **app-charts/aioc-hospital-{service}/** – one chart per service; each depends on the base
- **infra-charts/** (at repo root) – cluster infra (e.g. Promstack); see repo root for layout

## Install

From the repo root. The base chart path is `file://../aioc-hospital-base` (sibling under app-charts).

```bash
# Build dependencies (fetches/packages the base chart)
helm dependency build app-charts/aioc-hospital-login-service

# Install with release name = service name (so DNS is login-service, management-service, etc.)
helm install login-service app-charts/aioc-hospital-login-service -n aioc-hospital --create-namespace
helm install management-service app-charts/aioc-hospital-management-service -n aioc-hospital
helm install scheduling-service app-charts/aioc-hospital-scheduling-service -n aioc-hospital
helm install reports-service app-charts/aioc-hospital-reports-service -n aioc-hospital
helm install pdf-service app-charts/aioc-hospital-pdf-service -n aioc-hospital
helm install frontend app-charts/aioc-hospital-frontend -n aioc-hospital
```

## Before installing

1. **Images:** Set `aioc-hospital-base.image.repository` and `tag` (e.g. to your GHCR or registry). Defaults use `ghcr.io/your-org/<service>`.
2. **Gateway:** HTTPRoutes reference `main-gateway` in `kube-system`. Create that Gateway if you use Gateway API.
3. **Hostname:** Charts use subdomains under `aioc-services.com` (see below). Override in values if needed.
4. **Secrets:** Override `env` (e.g. `DATABASE_URL`, `SECRET_KEY`) via `--set` or a custom values file; avoid committing secrets.

## Host-based routing (aioc-services.com)

Each service is exposed on its own subdomain:

| Hostname | Service |
|----------|---------|
| my-hospital.aioc-services.com | frontend |
| login.aioc-services.com | login-service |
| management.aioc-services.com | management-service |
| scheduling.aioc-services.com | scheduling-service |
| reports.aioc-services.com | reports-service |
| pdf.aioc-services.com | pdf-service |

Point DNS A (or CNAME) records for each hostname to your Gateway's external IP.
