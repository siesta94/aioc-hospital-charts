# Infra Applications

Add Application manifests here for cluster infrastructure (e.g. monitoring, cert-manager).
The parent app `aioc-hospital-infra` (app-of-apps-infra) syncs this directory and creates them.

Example: add `promstack.yaml` pointing at `infra-charts/promstack` when you have that chart.
