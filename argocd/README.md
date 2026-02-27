# Argo CD Applications

Applications for the AIOC Hospital services. Charts live in this repo: **app-charts/** (application services + base) and **infra-charts/** (cluster infra, e.g. monitoring). Choose one of two ways to deploy apps.

## Two ways to deploy

### 1. Single environment (flat Application manifests)

Use the manifests in `argocd/applications/`. All apps go to one namespace: `aioc-hospital`.

- Apply all: `kubectl apply -f argocd/applications/`
- Apply one: `kubectl apply -f argocd/applications/frontend.yaml`

### 2. Multiple environments (ApplicationSet)

Use `application-set.yaml` to generate one Application per **service × environment**. Environments are listed in the ApplicationSet (default: `dev`, `staging`, `prod`). Each environment gets its own namespace: `aioc-hospital-dev`, `aioc-hospital-staging`, `aioc-hospital-prod`.

- Apply: `kubectl apply -f argocd/application-set.yaml`
- Argo CD will create Applications like `aioc-hospital-frontend-dev`, `aioc-hospital-frontend-staging`, etc.

Use **either** the apps in `applications/` **or** the ApplicationSet for a given cluster, so you don’t deploy the same chart twice into the same namespace.

## Setup

1. Ensure Argo CD has access to this repo (add it in **Settings → Repositories** if private).
2. Sync policy is **automated** (prune + self-heal). To use manual sync, remove the `syncPolicy.automated` block from the Application or ApplicationSet template.

## Customization

- **Namespace (single-env)**: Change `spec.destination.namespace` in the Application (default: `aioc-hospital`). `CreateNamespace=true` creates it if missing.
- **Environments (ApplicationSet)**: Edit `argocd/application-set.yaml` and change the `elements` under the second list generator (e.g. add `- env: qa` or remove `staging`).
- **Branch/tag**: Set `spec.source.targetRevision` to a branch (e.g. `main`) or tag instead of `HEAD`.
- **Values per environment**: In the ApplicationSet template you can add `spec.source.helm.values` or `valueFiles` and use `{{env}}` (e.g. a values file `values-{{env}}.yaml` in each chart) to vary config by environment.
