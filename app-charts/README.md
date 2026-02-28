# AIOC Hospital app charts

Application Helm charts live here. Each service chart uses **aioc-hospital-base** as a dependency. The base chart provides Deployment, Service, ServiceAccount, optional HTTPRoute (Gateway API), and supports `env` / `envFrom`.

## Layout

- **app-charts/aioc-hospital-base/** – base chart with shared templates
- **app-charts/aioc-hospital-{service}/** – one chart per service; each depends on the base
- **infra-charts/** (at repo root) – cluster infra; see repo root for layout

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

## Configuring domains

### 1. Set the hostname per service

Each service chart has **`httpRoute.hostnames`** in its `values.yaml`. That value is the domain (or subdomain) used for HTTP(S) routing. Edit the chart you care about:

| Service | File | Value to change |
|---------|------|------------------|
| Frontend | `app-charts/aioc-hospital-frontend/values.yaml` | `httpRoute.hostnames`: e.g. `- app.mycompany.com` |
| Login | `app-charts/aioc-hospital-login-service/values.yaml` | `httpRoute.hostnames`: e.g. `- login.mycompany.com` |
| Management | `app-charts/aioc-hospital-management-service/values.yaml` | same pattern |
| Scheduling | `app-charts/aioc-hospital-scheduling-service/values.yaml` | same pattern |
| Reports | `app-charts/aioc-hospital-reports-service/values.yaml` | same pattern |
| PDF | `app-charts/aioc-hospital-pdf-service/values.yaml` | same pattern |

Example for your own domain:

```yaml
httpRoute:
  enabled: true
  hostnames:
    - my-hospital.mycompany.com   # one hostname per service, or a list if you need several
```

Re-run `helm upgrade` (or your deploy process) to apply HTTPRoute changes. No need to touch the base chart unless you want to change the Gateway (`parentRefs`).

### 2. Point DNS at the Gateway

Your HTTPRoutes attach to the **Gateway** in the cluster (e.g. `main-gateway` in `kube-system`). Traffic must reach that Gateway’s external IP.

1. Get the Gateway’s external IP or hostname:
   ```bash
   kubectl get svc -n kube-system -l app.kubernetes.io/name=cilium-gateway-nfp
   # or whatever exposes the Gateway (e.g. LoadBalancer or Ingress)
   kubectl get gateway -n kube-system main-gateway -o wide
   ```
   Use the **EXTERNAL-IP** (or the hostname of the LoadBalancer) of the Service that backs the Gateway.

2. In your DNS provider (Cloudflare, Route53, etc.), create **A** or **CNAME** records:
   - **A record**: host = `my-hospital` (or `login`, etc.), value = **Gateway external IP**.
   - **CNAME**: host = `my-hospital`, value = **Gateway LB hostname** (if your provider gives one).

Do this for every hostname you set in step 1 (e.g. `my-hospital.mycompany.com`, `login.mycompany.com`, …). All of them can point to the **same** Gateway IP/hostname; the Gateway uses the `Host` header to route to the right service.

### 3. (Optional) HTTPS with cert-manager

You already have cert-manager and a ClusterIssuer (`letsencrypt-prod-dns`). To serve HTTPS:

- Configure the **Gateway** (or the Gateway’s listener) to use TLS and reference a **Certificate** in the same namespace as the Gateway (e.g. `kube-system`). The Certificate should use the same hostnames as your HTTPRoutes and the `letsencrypt-prod-dns` ClusterIssuer (DNS-01 is typical for wildcards).
- Ensure the DNS-01 secret (e.g. Cloudflare API token) exists for your cert-manager ClusterIssuer if you use TLS.

Once the Certificate is issued, the Gateway will serve TLS for those hostnames.

---

## Host-based routing (default: aioc-services.com)

Default hostnames in the charts (you can override as above):

| Hostname | Service |
|----------|---------|
| my-hospital.aioc-services.com | frontend |
| login.aioc-services.com | login-service |
| management.aioc-services.com | management-service |
| scheduling.aioc-services.com | scheduling-service |
| reports.aioc-services.com | reports-service |
| pdf.aioc-services.com | pdf-service |

Point DNS A (or CNAME) records for each hostname to your Gateway’s external IP.
