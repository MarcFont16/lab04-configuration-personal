Lab03 - Definition of Done

Entrypoint and exposure

- Traefik is the single public entrypoint for the application.
- Frontend and backend are exposed internally as `ClusterIP` services.
- Public traffic is routed through Kubernetes Ingress with `ingressClassName: traefik`.
- Path-based routing is active on one host:
  - `/` -> frontend
  - `/api` -> backend
- No unnecessary public `LoadBalancer` service exists for application components.

DNS and TLS

- The public host is the course DNS `g02.cpo2026.it`.
- Course DNS is mapped to the Azure Traefik FQDN.
- TLS is managed by cert-manager with Let's Encrypt.
- Certificate flow is validated with staging first, then production.
- Active issuer is `letsencrypt-prod`.
- HTTP is redirected to HTTPS.

CloudNativePG and PostgreSQL

- CloudNativePG operator is installed once in `cnpg-system`.
- One PostgreSQL `Cluster` resource exists in `lab03-app` and reaches healthy state.
- Operator-generated database services are present:
  - `my-secure-app-cluster-rw`
  - `my-secure-app-cluster-ro`
  - `my-secure-app-cluster-r`
- Operator-generated credentials secret is present:
  - `my-secure-app-cluster-app`

Backend runtime configuration

- Backend runs with `SPRING_PROFILES_ACTIVE=postgres`.
- Backend database target is `my-secure-app-cluster-rw`.
- `DB_PORT` is set to `5432`.
- `DB_NAME`, `DB_USER`, and `DB_PASS` are injected from `my-secure-app-cluster-app`.

Submission

- Demo video link is published in `README.md` or `docs/lab03-demo.md`.
