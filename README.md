## Kubernetes Deployment and Operations Guide (Lab03)

This repository defines the deployment-side state for Lab03: exposing and securing a multi-component application on AKS with Traefik, cert-manager, and CloudNativePG.

## 1. Scope

This repository covers:
- Traefik as single public ingress entrypoint
- Ingress path-based routing (`/` and `/api`)
- TLS automation with cert-manager and Let's Encrypt (staging then production)
- CloudNativePG operator-managed PostgreSQL dependency
- Runtime wiring of backend to operator-managed database service and secret
- DoD and demo evidence documentation

## 2. Repository Ownership Model

- **Application repository** owns source code, tests, CI pipelines, and image build/publish logic.
- **Configuration repository** owns deployment configuration, platform manifests, runtime exposure rules, and operational evidence.

## 3. Repository Structure

Relevant paths:
- `deploy/charts/my-secure-app/`
  - `Chart.yaml`
  - `values.yaml`
  - `templates/` (frontend/backend deployments and services, ingress)
- `platform/traefik/`
  - `values.yaml`
  - `middleware.yaml`
- `platform/cert-manager/`
  - `letsencrypt-staging.yaml`
  - `letsencrypt-prod.yaml`
- `docs/lab03-dod.md`
- `docs/lab03-demo.md`

## 4. Runtime Baseline

Target runtime:
- AKS cluster (shared group cluster)
- Namespace: `lab03-app`
- Release: `my-secure-app`
- Public host: `g02.cpo2026.it`
- Ingress class: `traefik`

## 5. Platform Components

### 5.1 Traefik

Traefik is installed in namespace `traefik` and exposed as the only public `LoadBalancer`.

### 5.2 cert-manager

cert-manager is installed in namespace `cert-manager` and manages ACME HTTP-01 certificate issuance using:
- `letsencrypt-staging` (validation phase)
- `letsencrypt-prod` (production phase)

### 5.3 CloudNativePG

CloudNativePG operator is installed in namespace `cnpg-system`.
The application chart includes CNPG dependency `cluster` and deploys PostgreSQL cluster resources in `lab03-app`.

## 6. Application Exposure Model

- Frontend service type: `ClusterIP`
- Backend service type: `ClusterIP`
- Ingress host: `g02.cpo2026.it`
- Routing:
  - `/` -> frontend (`my-secure-app-app-chart-frontend`)
  - `/api` -> backend (`my-secure-app-app-chart-backend`)
- HTTP to HTTPS redirect is enabled through Traefik middleware annotation.

## 7. Database Wiring Model

Backend runtime wiring is expected to be:
- `SPRING_PROFILES_ACTIVE=postgres`
- `DB_HOST=my-secure-app-cluster-rw`
- `DB_PORT=5432`
- `DB_NAME`, `DB_USER`, `DB_PASS` from secret `my-secure-app-cluster-app`

Operator-generated DB services:
- `my-secure-app-cluster-rw` (read/write primary target)
- `my-secure-app-cluster-ro` (read-only traffic when replicas are available)
- `my-secure-app-cluster-r` (generic read traffic)

## 8. Deployment Commands

### 8.1 Chart dependency resolution

```bash
helm dependency update ./deploy/charts/my-secure-app
```

### 8.2 Install or upgrade

```bash
helm upgrade --install my-secure-app ./deploy/charts/my-secure-app \
  --namespace lab03-app \
  --create-namespace
```

## 9. Operational Validation Commands

### 9.1 Entrypoint and routing

```bash
kubectl get pods,svc -n traefik
kubectl get ingressclass
kubectl get svc -A --field-selector spec.type=LoadBalancer
kubectl get svc,endpointslice -n lab03-app
kubectl describe ingress -n lab03-app my-secure-app
```

### 9.2 DNS and HTTPS

```bash
nslookup g02.cpo2026.it
curl -I http://g02.cpo2026.it/
curl -I https://g02.cpo2026.it/
```

### 9.3 cert-manager status

```bash
kubectl get clusterissuer
kubectl get certificate,certificaterequest,order,challenge -n lab03-app
kubectl describe certificate -n lab03-app my-secure-app-tls
```

### 9.4 CloudNativePG and backend DB wiring

```bash
kubectl get pods -n cnpg-system
kubectl get cluster,pods,pvc,svc,secrets -n lab03-app
kubectl get deploy -n lab03-app my-secure-app-app-chart-backend -o yaml
```

## 10. The demo video 

The demo video link in `docs/lab03-demo.md`.
