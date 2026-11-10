# Argo CD GitOps Continuous Deployment & Kubernetes Observability

This project demonstrates an enterprise-oriented **Argo CD-based GitOps continuous deployment and Kubernetes observability architecture** for the **Resume Matcher** platform, integrating declarative application delivery, automated GitOps synchronization, Kubernetes cluster management, immutable container image deployment, deployment verification, infrastructure monitoring, application telemetry, Prometheus alerting, Grafana dashboards, and automated email notifications.

The implementation extends the existing DevSecOps delivery lifecycle by introducing **Argo CD as the GitOps continuous deployment and reconciliation layer**, enabling Kubernetes workloads to be managed declaratively from Git repositories.

The architecture integrates GitHub Actions ephemeral runners, zrok secure tunnelling, SSH port forwarding, Kubernetes, Argo CD, GitHub Container Registry (GHCR), Prometheus, Grafana, and Alertmanager.

The deployment lifecycle connects previously established CI/CD and supply-chain security stages with Kubernetes continuous deployment and operational observability.

The overall implementation covers:

1. **Argo CD Infrastructure, zrok Tunnelling & Remote Access**
2. **Kubernetes Cluster & Argo CD Installation**
3. **GitHub Repository Integration & Credential Management**
4. **GHCR Integration, Kubernetes Secrets & Deployment Verification**
5. **Declarative GitOps Application & Automated Synchronization**
6. **Prometheus & Grafana Monitoring Infrastructure**
7. **Application Telemetry, Metrics Scraping & Dashboard Visualization**
8. **Prometheus Alerting, Alertmanager & Email Notifications**

---

## Access the Walkthrough

[![Argo CD GitOps Continuous Deployment & Kubernetes Observability](https://img.youtube.com/vi/WspnhcoLXOg/0.jpg)](https://www.youtube.com/embed/WspnhcoLXOg?si=ZAi8GgHKIOaHHhVP)

[Watch the Argo CD GitOps Continuous Deployment & Kubernetes Observability Walkthrough](https://www.youtube.com/embed/WspnhcoLXOg?si=ZAi8GgHKIOaHHhVP)

---

# 🏗️ Argo CD GitOps Deployment Strategy

The Argo CD implementation represents the Kubernetes continuous deployment and observability stage of the broader DevSecOps pipeline.

It takes over after the completion of application validation, security scanning, container publishing, and supply-chain security processes.

The upstream delivery architecture includes:

```text
GitHub Repository
       │
       ▼
GitHub Actions
       │
       ▼
PRE-MAIN VALIDATION
       │
       ├── Jest
       ├── ESLint
       ├── SonarCloud
       ├── Gitleaks
       ├── OWASP Dependency-Check
       ├── Trivy
       ├── Cypress
       ├── WireMock
       └── k6
       │
       ▼
PRODUCTION RELEASE
       │
       ├── Docker Buildx
       ├── Multi-Architecture Build
       ├── GHCR Publishing
       ├── Docker Hub Publishing
       ├── Immutable SHA-256 Digest
       ├── Cosign Signing
       └── Syft SPDX SBOM
       │
       ▼
GITOPS CONTINUOUS DEPLOYMENT
       │
       ▼
     Argo CD
       │
       ├── Git Repository Integration
       ├── Declarative Application
       ├── Automated Synchronization
       ├── Self-Healing
       ├── Kubernetes Reconciliation
       ├── Deployment Verification
       └── Immutable Image Validation
       │
       ▼
KUBERNETES OBSERVABILITY
       │
       ├── Prometheus
       ├── Grafana
       ├── Alertmanager
       ├── ServiceMonitors
       ├── Application Metrics
       ├── Kubernetes Metrics
       └── Email Alert Notifications
```

The architecture separates container production and security verification from Kubernetes deployment management and operational monitoring.

---

# Step 1: Argo CD Infrastructure, zrok Tunnelling & Remote Access

The first stage establishes the remote infrastructure management environment required to operate Kubernetes and Argo CD.

Instead of depending on continuously hosted cloud infrastructure such as AWS, the implementation leverages GitHub Actions runners as ephemeral remote compute environments.

The architecture combines:

- GitHub Actions runners
- Ubuntu remote execution environment
- zrok secure tunnelling
- SSH port forwarding
- Local Windows terminal access
- Remote Kubernetes management

## GitHub Actions & Remote Infrastructure Architecture

GitHub Actions provides the remote execution environment, while zrok exposes the required connectivity through a secure tunnel.

The resulting architecture is:

```text
Local Windows Machine
        │
        ▼
     zrok CLI
        │
        ▼
 Private zrok Tunnel
        │
        ▼
    zrok Proxy
        │
        ▼
 GitHub Actions Runner
        │
        ▼
   Remote Ubuntu Host
        │
        ▼
    SSH Connection
        │
        ▼
 Port Forwarding
        │
        ├── 8080 → Argo CD
        └── 3000 → Grafana
```

The SSH tunnel provides local access to remote infrastructure services without requiring direct public exposure of their internal service ports.

### zrok Environment Initialization

The zrok CLI is initialized using the account token.

```bash
# Enable zrok account environment
zrok enable <zrok-token>
```

### Private zrok Tunnel Access

A private zrok share is accessed locally through port 9000.

```bash
# Establish local private tunnel endpoint
zrok.exe access private --bind 127.0.0.1:9000 <share-token>
```

### SSH Port Forwarding

The SSH connection forwards the remote Argo CD and Grafana service ports to the local Windows environment.

```bash
ssh -o StrictHostKeyChecking=no \
    -o UserKnownHostsFile=/dev/null \
    -L 8080:localhost:8080 \
    -L 3000:localhost:8081 \
    root@127.0.0.1 -p 9000
```

The resulting connections allow access to remote infrastructure services through locally accessible endpoints.

**Important:** The six-hour runner lifecycle represents the configured temporary execution window in this implementation, not a permanent hosting environment.

### Implementation Evidence

The following screenshots document the architecture initialization, zrok configuration, GitHub Actions execution, and SSH connectivity.

<div align="center">

<img src="images/argocd/argo 1/argo1.1.png" width="1000"/>

<img src="images/argocd/argo 1/argo1.2.png" width="1000"/>

<img src="images/argocd/argo 1/argo1.3.png" width="450"/>
<img src="images/argocd/argo 1/argo1.4.png" width="450"/>

<img src="images/argocd/argo 1/argo1.5.png" width="450"/>
<img src="images/argocd/argo 1/argo1.6.png" width="450"/>

<img src="images/argocd/argo 1/argo1.7.png" width="450"/>
<img src="images/argocd/argo 1/argo1.8.png" width="450"/>

<img src="images/argocd/argo 1/argo1.9.png" width="450"/>

</div>

---

# Step 2: Kubernetes Cluster & Argo CD Installation

The second stage establishes the Kubernetes environment and deploys the Argo CD continuous deployment control plane.

The implementation uses **Kind Kubernetes** to provision the cluster and the official Argo CD installation manifests to deploy its components.

## Kubernetes Cluster Initialization

The Kind cluster is created and its connectivity is verified.

```bash
# Create Kubernetes cluster
kind create cluster

# Verify cluster connectivity
kubectl cluster-info
```

## Argo CD Installation

Argo CD is installed into a dedicated Kubernetes namespace.

```bash
# Create Argo CD namespace
kubectl create namespace argocd

# Install official Argo CD manifests
kubectl apply --server-side \
  -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify Argo CD components
kubectl get pods -n argocd
```

The installation deploys the Argo CD control-plane components responsible for repository access, application management, synchronization, and deployment reconciliation.

## Argo CD Administrator Authentication

The initial administrator password is retrieved from the Kubernetes secret.

```bash
# Retrieve initial administrator password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

## Argo CD Web Interface Access

The Argo CD server is exposed locally through Kubernetes port forwarding.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

The dashboard is accessed through:

```text
https://localhost:8080
```

Authentication uses:

```text
Username: admin
Password: Initial Argo CD administrator password
```

Following authentication, the Argo CD dashboard provides visibility into registered repositories, configured applications, deployment health, and synchronization status.

The implementation also includes reviewing the `argocd-app.yaml` declarative application manifest and applying the initial application definition.

### Implementation Evidence

<div align="center">

<img src="images/argocd/argo 2/argo2.1.png" width="450"/>
<img src="images/argocd/argo 2/argo2.2.png" width="450"/>

<img src="images/argocd/argo 2/argo2.3.png" width="450"/>
<img src="images/argocd/argo 2/argo2.4.png" width="450"/>

<img src="images/argocd/argo 2/argo2.5.jpg" width="450"/>
<img src="images/argocd/argo 2/argo2.6.png" width="450"/>

<img src="images/argocd/argo 2/argo2.7.png" width="450"/>
<img src="images/argocd/argo 2/argo2.8.png" width="450"/>

</div>

---

# Step 3: GitHub Repository Integration & Credential Management

The third stage establishes secure communication between Argo CD, GitHub, and automated deployment verification processes.

GitHub repository authentication is configured through fine-grained Personal Access Tokens (PATs).

These credentials allow Argo CD to access the Kubernetes manifest repository and allow supporting automation to interact with the required repository and deployment verification interfaces.

## GitHub Repository Registration

The GitHub repository is registered with Argo CD through its CLI.

```bash
argocd repo add \
  https://github.com/kaifmohammedkhan/resume-matcher-devops.git \
  --username kaifmohammedkhan \
  --password <github-personal-access-token>
```

The registered repository is verified using:

```bash
argocd repo list
```

## Argo CD API Authentication

An Argo CD service account named:

```text
pipeline-verifier
```

is configured for automated pipeline verification.

The associated API authentication token is generated through Argo CD account settings.

The required authentication information is stored in GitHub repository secrets.

```text
ARGOCD_AUTH_TOKEN
ARGOCD_SERVER
```

These secrets support authenticated interactions with the Argo CD control plane.

## Credential Integration Architecture

```text
GitHub Repository
       │
       ▼
Fine-Grained GitHub PAT
       │
       ▼
Argo CD Repository Integration
       │
       ▼
GitOps Manifest Access
       │
       ▼
Argo CD Application Controller
       │
       ▼
Kubernetes Deployment

GitHub Actions
       │
       ▼
ARGOCD_AUTH_TOKEN
       │
       ▼
pipeline-verifier
       │
       ▼
Argo CD API
```

The credential model separates repository authentication from Argo CD API authentication.

### Implementation Evidence

<div align="center">

<img src="images/argocd/argo 3/argo3.1.png" width="450"/>
<img src="images/argocd/argo 3/argo3.2.png" width="450"/>

<img src="images/argocd/argo 3/argo3.3.png" width="450"/>
<img src="images/argocd/argo 3/argo3.4.png" width="450"/>

<img src="images/argocd/argo 3/argo3.5.png" width="450"/>
<img src="images/argocd/argo 3/argo3.6.png" width="450"/>

<img src="images/argocd/argo 3/argo3.7.png" width="450"/>

</div>

---

# Step 4: GHCR Integration, Kubernetes Secrets & Deployment Verification

The fourth stage connects the Kubernetes environment with GitHub Container Registry and introduces deployment verification capabilities.

The objective is to ensure that Kubernetes workloads use the expected production container image and that the deployed application reflects the intended immutable image identity.

## GHCR Authentication & Kubernetes Secret Provisioning

A dedicated automation script:

```text
ghcr.sh
```

is used to authenticate against the container registry and provision the required Kubernetes secrets.

The implementation includes:

- GHCR registry authentication
- `ghcr-secret` Kubernetes image-pull secret
- Verifier authentication secrets
- Alerting credentials
- Argo CD RBAC configuration

Execute the helper script:

```bash
chmod +x ghcr.sh && ./ghcr.sh
```

## Argo CD RBAC Configuration

The Argo CD RBAC configuration is applied to establish the required access controls.

```bash
kubectl apply -f argocd-rbac-setup.yaml
```

## Deployment Verifier Job

The deployment verifier manifest is applied to the Argo CD namespace.

```bash
kubectl apply -f argocd-deployment-verifier.yaml
```

The verifier job provides an additional validation layer for the live Kubernetes deployment.

It checks:

- Expected deployment image identity
- Immutable SHA-256 image digest
- Kubernetes deployment synchronization state
- Application health
- Desired and ready replica counts
- Deployed container image references

The documented deployment validation includes a three-replica readiness target.

```text
Expected Replicas: 3
Ready Replicas:    3
Image Identity:    Expected SHA-256 Digest
```

## Deployment Verification Logs

The verifier execution can be inspected through Kubernetes logs.

```bash
kubectl logs job/argocd-deployment-verifier-job \
  -f -n argocd
```

## Deployment Verification Architecture

```text
GitHub Container Registry
          │
          ▼
    Immutable Digest
          │
          ▼
    Kubernetes Manifests
          │
          ▼
        Argo CD
          │
          ▼
    Kubernetes Cluster
          │
          ▼
  Deployment Verifier Job
          │
          ├── Application Health
          ├── Sync Status
          ├── Replica Availability
          ├── Container Image
          └── SHA-256 Digest
          │
          ▼
   Verification Evidence
          │
          ▼
      JSON Output
```

This additional verification layer provides deployment-level evidence beyond simply confirming that an Argo CD application has synchronized.

### Implementation Evidence

<div align="center">

<img src="images/argocd/argo 4/argo4.1.png" width="450"/>
<img src="images/argocd/argo 4/argo4.2.png" width="450"/>

<img src="images/argocd/argo 4/argo4.3.png" width="900"/>

</div>

---

# Step 5: Declarative GitOps Application & Automated Synchronization

The fifth stage introduces the declarative GitOps application definition.

Argo CD continuously manages the relationship between the desired Kubernetes configuration stored in Git and the actual resources running inside the target Kubernetes cluster.

The implementation uses the Argo CD `Application` custom resource.

## Argo CD Application Manifest

The application definition specifies:

- Application name
- Argo CD project
- Git repository
- Target Git revision
- Kubernetes manifest directory
- Destination cluster
- Destination namespace
- Automated synchronization
- Resource pruning
- Self-healing
- Namespace creation

The application manifest is:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: resume-matcher-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/kaifmohammedkhan/resume-matcher-devops.git
    targetRevision: HEAD
    path: k8s/manifests

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=true
```

## GitOps Synchronization Architecture

```text
GitHub Repository
       │
       ▼
Kubernetes Manifests
       │
       ▼
Argo CD Repository Server
       │
       ▼
Application Controller
       │
       ▼
Desired State Comparison
       │
       ▼
Kubernetes Cluster
       │
       ├── Automated Sync
       ├── Resource Pruning
       ├── Self-Healing
       └── Namespace Management
       │
       ▼
Actual Cluster State
```

## Automated Synchronization

The configuration enables three important GitOps capabilities.

**Automated Sync**

Argo CD automatically synchronizes the desired configuration from Git with the Kubernetes cluster.

**Pruning**

The `prune: true` configuration allows Argo CD to remove resources that are no longer defined in the desired application state.

**Self-Healing**

The `selfHeal: true` configuration allows Argo CD to reconcile live cluster changes that diverge from the desired Git configuration.

**Namespace Creation**

The `CreateNamespace=true` sync option allows Argo CD to create the destination namespace when required.

## Immutable Image Deployment Verification

The application deployment is additionally inspected against the production image published to GHCR.

The implementation verifies that Kubernetes workloads use the expected digest-pinned image rather than relying exclusively on mutable image tags.

The validation includes:

- GHCR package inspection
- SHA-256 digest comparison
- Kubernetes pod image inspection
- Container image ID verification
- Application availability
- Pod execution logs

### Implementation Evidence

<div align="center">

<img src="images/argocd/argo 5/argo5.1.png" width="450"/>
<img src="images/argocd/argo 5/argo5.2.png" width="450"/>

<img src="images/argocd/argo 5/argo5.3.png" width="450"/>
<img src="images/argocd/argo 5/argo5.4.png" width="450"/>

<img src="images/argocd/argo 5/argo5.5.png" width="450"/>
<img src="images/argocd/argo 5/argo5.6.png" width="450"/>

</div>

---

# Step 6: Prometheus & Grafana Monitoring Infrastructure

The sixth stage introduces the Kubernetes observability infrastructure.

Prometheus and Grafana are deployed using the official Prometheus Community Helm chart.

The monitoring architecture provides visibility into cluster resources, application telemetry, infrastructure utilization, and operational health.

## Prometheus Community Helm Repository

First, the Prometheus Community Helm repository is configured.

```bash
# Add Prometheus Community repository
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

# Update Helm repositories
helm repo update
```

## Monitoring Namespace

A dedicated Kubernetes namespace is created.

```bash
kubectl create namespace monitoring
```

## kube-prometheus-stack Installation

The monitoring stack is deployed using Helm.

```bash
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring
```

The installation establishes the core monitoring components:

```text
kube-prometheus-stack
        │
        ├── Prometheus
        ├── Alertmanager
        ├── Grafana
        ├── Prometheus Operator
        └── Node Exporters
```

## Grafana Administrator Authentication

The Grafana administrator password is retrieved from the Kubernetes secret.

```bash
kubectl get secret \
  --namespace monitoring monitoring-grafana \
  -o jsonpath="{.data.admin-password}" \
  | base64 --decode; echo
```

## Grafana Port Forwarding

Grafana is exposed locally using:

```bash
kubectl --namespace monitoring \
  port-forward svc/monitoring-grafana 3001:80
```

The Grafana interface is accessed through:

```text
http://localhost:3001
```

## Prometheus Port Forwarding

Prometheus is exposed using:

```bash
kubectl port-forward \
  svc/monitoring-kube-prometheus-prometheus \
  9090:9090 \
  -n monitoring
```

The Prometheus interface is accessed through:

```text
http://localhost:9090
```

## Monitoring Architecture

```text
Kubernetes Cluster
       │
       ▼
kube-prometheus-stack
       │
       ├── Prometheus Operator
       │
       ├── Prometheus
       │      │
       │      ├── Kubernetes Metrics
       │      ├── Node Metrics
       │      └── Application Metrics
       │
       ├── Alertmanager
       │
       ├── Grafana
       │
       └── Node Exporters
                │
                ▼
         Metrics Collection
                │
                ▼
         Grafana Dashboards
```

## Grafana Dashboard Configuration

The Grafana environment is configured to use Prometheus and Alertmanager as monitoring data sources.

The implementation also includes importing dashboards for:

- Node Exporter Full
- Kubernetes cluster resources
- Pod-level resource utilization
- CPU utilization
- Memory utilization
- Container and workload metrics

### Implementation Evidence

<div align="center">

<img src="images/argocd/argo 6/argo6.1.png" width="450"/>
<img src="images/argocd/argo 6/argo6.2.png" width="450"/>

<img src="images/argocd/argo 6/argo6.3.png" width="450"/>
<img src="images/argocd/argo 6/argo6.4.png" width="450"/>

<img src="images/argocd/argo 6/argo6.5.jpg" width="450"/>
<img src="images/argocd/argo 6/argo6.6.png" width="450"/>

<img src="images/argocd/argo 6/argo6.7.jpg" width="450"/>
<img src="images/argocd/argo 6/argo6.8.png" width="450"/>

<img src="images/argocd/argo 6/argo6.9.png" width="450"/>
<img src="images/argocd/argo 6/argo6.10.png" width="450"/>

</div>

---

# Step 7: Application Telemetry, Metrics Scraping & Grafana Dashboard Visualization

The seventh stage extends infrastructure monitoring with application-level observability.

The Resume Matcher application exposes Prometheus-compatible metrics through its `/metrics` endpoint.

Prometheus ServiceMonitors are configured to discover and scrape application metrics.

## Application Metrics Endpoint

The application metrics endpoint is accessible through:

```bash
curl http://localhost:3000/metrics
```

The endpoint exposes application and runtime telemetry.

The documented monitoring categories include:

```text
Application Metrics
       │
       ├── CPU Utilization
       ├── Memory Utilization
       ├── Event Loop Lag
       └── File Descriptor Metrics
```

## Prometheus ServiceMonitor Integration

Prometheus uses Kubernetes ServiceMonitors to discover configured application monitoring targets.

The monitoring workflow includes:

1. Exposing the application metrics endpoint.
2. Configuring the Kubernetes monitoring resources.
3. Enabling Prometheus target discovery.
4. Scraping application metrics.
5. Verifying target availability.
6. Visualizing metrics through Grafana.

## Prometheus Target Verification

Prometheus target discovery is verified through the Prometheus interface.

The target status page provides visibility into the application monitoring endpoint and its scraping status.

The implementation verifies active `UP` scrape targets for:

```text
resume-matcher-monitor
```

## Grafana Application & Kubernetes Dashboards

Grafana provides visual representations of the collected metrics.

The configured dashboard views include:

- Node-level CPU utilization
- Memory utilization
- Disk utilization
- Kubernetes cluster resources
- Total pod count
- Workload resource consumption
- Individual pod CPU usage
- Pod working-set memory
- Multi-pod resource comparison

## Observability Architecture

```text
Resume Matcher Application
          │
          ▼
      /metrics
          │
          ▼
  Kubernetes ServiceMonitor
          │
          ▼
      Prometheus
          │
          ├── Application Metrics
          ├── CPU Metrics
          ├── Memory Metrics
          ├── Event Loop Metrics
          └── File Descriptor Metrics
          │
          ▼
       Grafana
          │
          ├── Node Dashboards
          ├── Cluster Dashboards
          ├── Pod Dashboards
          └── Application Monitoring
```

The observability layer provides visibility into both application runtime behavior and Kubernetes infrastructure utilization.

### Implementation Evidence

<div align="center">

<img src="images/argocd/argo 7/argo7.1.png" width="450"/>
<img src="images/argocd/argo 7/argo7.2.png" width="450"/>

<img src="images/argocd/argo 7/argo7.3.png" width="450"/>
<img src="images/argocd/argo 7/argo7.4.png" width="450"/>

<img src="images/argocd/argo 7/argo7.5.png" width="450"/>
<img src="images/argocd/argo 7/argo7.6.png" width="450"/>

<img src="images/argocd/argo 7/argo7.7.png" width="450"/>

</div>

---

# Step 8: Prometheus Alerting Rules, Alertmanager Configuration & Email Notifications

The eighth stage introduces proactive monitoring and automated incident notifications.

Prometheus alerting rules are configured to identify operational issues affecting the application and Kubernetes infrastructure.

Alertmanager receives firing alerts from Prometheus and routes notifications through the configured SMTP email receiver.

The implementation demonstrates alert generation by deliberately scaling the Resume Matcher deployment to zero replicas.

## Prometheus Alerting Architecture

```text
Kubernetes Cluster
        │
        ▼
     Prometheus
        │
        ▼
   Alerting Rules
        │
        ├── Service Health
        ├── Scraping Failures
        ├── High Latency
        └── Zero Replicas
        │
        ▼
   Alertmanager
        │
        ▼
    SMTP Receiver
        │
        ▼
   Email Notification
        │
        ▼
       Gmail
```

## Zero-Replica Alert Testing

To explicitly test the alerting workflow, the Resume Matcher deployment is scaled down to zero replicas.

```bash
kubectl scale deployment resume-matcher-ghcr \
  --replicas=0 \
  -n default
```

This triggers the configured:

```text
ResumeMatcherZeroReplicas
```

alert when its Prometheus rule conditions are satisfied.

The resulting alert demonstrates the integration between Kubernetes workload state, Prometheus alert evaluation, Alertmanager routing, and email notifications.

## Alertmanager Configuration Inspection

The active Alertmanager configuration is inspected from its Kubernetes pod.

```bash
kubectl exec -it \
  alertmanager-monitoring-kube-prometheus-alertmanager-0 \
  -n monitoring \
  -c alertmanager -- \
  cat /etc/alertmanager/config_out/alertmanager.env.yaml
```

The configuration includes the SMTP receiver and email routing information.

## Alertmanager Logs

The Alertmanager logs are inspected to verify configuration loading and notification dispatch events.

```bash
kubectl logs \
  alertmanager-monitoring-kube-prometheus-alertmanager-0 \
  -n monitoring \
  -c alertmanager \
  --tail=50
```

## Alert Notification Lifecycle

```text
Deployment Replica Count = 0
              │
              ▼
       Kubernetes State
              │
              ▼
          Prometheus
              │
              ▼
      Alert Rule Evaluation
              │
              ▼
   ResumeMatcherZeroReplicas
              │
              ▼
         Alertmanager
              │
              ▼
       SMTP Configuration
              │
              ▼
       Email Notification
              │
              ▼
            Gmail
```

## Automated Incident Notification

The final stage demonstrates that Kubernetes deployment failures and availability conditions can be surfaced through automated notifications.

The implementation verifies the firing alert through Prometheus, inspects the Alertmanager configuration and runtime logs, and confirms delivery through Gmail.

### Implementation Evidence

<div align="center">

<img src="images/argocd/argo 8/argo8.1.png" width="450"/>
<img src="images/argocd/argo 8/argo8.2.png" width="450"/>

<img src="images/argocd/argo 8/argo8.3.png" width="450"/>
<img src="images/argocd/argo 8/argo8.4.png" width="450"/>

<img src="images/argocd/argo 8/argo8.5.png" width="450"/>
<img src="images/argocd/argo 8/argo8.6.png" width="450"/>

<img src="images/argocd/argo 8/argo8.7.png" width="450"/>
<img src="images/argocd/argo 8/argo8.8.png" width="450"/>

</div>

---

# 🔄 End-to-End Argo CD GitOps Deployment & Observability Architecture

The completed architecture connects the existing DevSecOps delivery pipeline with GitOps continuous deployment and Kubernetes observability.

```text
                 DEVELOPER
                    │
                    ▼
              GITHUB REPOSITORY
                    │
                    ▼
             GITHUB ACTIONS
                    │
          ┌─────────┴──────────┐
          ▼                    ▼
    PRE-MAIN CI           PRODUCTION CI/CD
          │                    │
          ├── Jest             ├── Docker Buildx
          ├── ESLint           ├── Multi-Arch Build
          ├── SonarCloud       ├── GHCR
          ├── Gitleaks         ├── Docker Hub
          ├── OWASP            ├── SHA-256 Digest
          ├── Trivy            ├── Cosign
          ├── Cypress          └── SPDX SBOM
          ├── WireMock               │
          └── k6                     ▼
                              VERIFIED ARTIFACTS
                                      │
                                      ▼
                               GITOPS DELIVERY
                                      │
                                      ▼
                                  zrok
                                      │
                                      ▼
                             REMOTE INFRASTRUCTURE
                                      │
                                      ▼
                                KUBERNETES
                                      │
                                      ▼
                                  ARGO CD
                                      │
                         ┌────────────┴────────────┐
                         ▼                         ▼
                  GITHUB REPOSITORY          GHCR IMAGE
                         │                         │
                         ▼                         ▼
                   Desired State             Immutable Image
                         │                         │
                         └────────────┬────────────┘
                                      │
                                      ▼
                             GITOPS SYNCHRONIZATION
                                      │
                                      ▼
                             KUBERNETES DEPLOYMENT
                                      │
                                      ▼
                             DEPLOYMENT VERIFICATION
                                      │
                                      ▼
                             APPLICATION RUNNING
                                      │
                                      ▼
                              OBSERVABILITY STACK
                                      │
                         ┌────────────┼────────────┐
                         ▼            ▼            ▼
                    Prometheus     Grafana    Alertmanager
                         │            │            │
                         ▼            ▼            ▼
                     Metrics     Dashboards    Alert Rules
                         │            │            │
                         └────────────┼────────────┘
                                      │
                                      ▼
                               SMTP NOTIFICATION
                                      │
                                      ▼
                                    GMAIL
```

---

# 🔐 GitOps & Kubernetes Security Controls

| Security / Operational Layer | Technology | Purpose |
|---|---|---|
| Source Control | GitHub | Version-controlled Kubernetes manifests |
| CI/CD Integration | GitHub Actions | Automated upstream delivery |
| Remote Execution | GitHub Actions Runners | Ephemeral infrastructure execution |
| Secure Ingress | zrok | Remote connectivity |
| Remote Administration | SSH | Secure port-forwarded access |
| Container Registry | GHCR | Production image distribution |
| Artifact Identity | SHA-256 Digest | Immutable image identification |
| GitOps Controller | Argo CD | Declarative continuous deployment |
| Repository Authentication | GitHub PAT | Git repository access |
| API Authentication | Argo CD Tokens | Automated Argo CD access |
| Access Control | Argo CD RBAC | Role-based permissions |
| Kubernetes Platform | Kind / Kubernetes | Container orchestration |
| Image Authentication | Kubernetes Secrets | Private registry access |
| Deployment Verification | Python Kubernetes Client | Live deployment validation |
| Monitoring | Prometheus | Metrics collection |
| Metrics Discovery | ServiceMonitor | Application metrics discovery |
| Visualization | Grafana | Monitoring dashboards |
| Infrastructure Monitoring | Node Exporter | Node resource telemetry |
| Alert Evaluation | Prometheus Rules | Operational alert detection |
| Alert Routing | Alertmanager | Alert notification management |
| Notifications | SMTP / Gmail | Automated incident notifications |

---

# 🧰 Argo CD GitOps & Observability Toolchain

| Category | Technology |
|---|---|
| Source Control | Git / GitHub |
| CI/CD Integration | GitHub Actions |
| Remote Compute | GitHub Actions Runners |
| Secure Tunnelling | zrok |
| Remote Access | SSH |
| Container Registry | GHCR |
| Container Identity | SHA-256 Digest |
| Container Orchestration | Kubernetes |
| Local Cluster | Kind |
| GitOps Continuous Deployment | Argo CD |
| Application Definition | Argo CD Application CRD |
| Repository Authentication | GitHub Fine-Grained PAT |
| API Authentication | Argo CD API Tokens |
| Access Management | Argo CD RBAC |
| Deployment Validation | Kubernetes Python Client |
| Package Management | Helm |
| Monitoring Stack | kube-prometheus-stack |
| Metrics Collection | Prometheus |
| Metrics Discovery | ServiceMonitor |
| Visualization | Grafana |
| Node Monitoring | Node Exporter |
| Alert Evaluation | Prometheus Alerting Rules |
| Alert Management | Alertmanager |
| Notification | SMTP / Gmail |
| Application Telemetry | Prometheus /metrics |
| Infrastructure | Kubernetes |
| Automation | Bash / Shell |

---

# 📦 Deployment & Observability Evidence

The implementation preserves visual evidence across eight major implementation stages.

## Infrastructure & Access Evidence

<img src="images/argocd/argo 1/argo1.6.png" width="450"/>
<img src="images/argocd/argo 1/argo1.7.png" width="450"/>
<img src="images/argocd/argo 1/argo1.8.png" width="450"/>
<img src="images/argocd/argo 1/argo1.9.png" width="450"/>

Documents zrok initialization, GitHub Actions remote execution, tunnel establishment, and SSH access.

## Argo CD Installation Evidence

<img src="images/argocd/argo 2/argo2.3.png" width="450"/>
<img src="images/argocd/argo 2/argo2.4.png" width="450"/>
<img src="images/argocd/argo 2/argo2.5.png" width="450"/>

Documents Kind cluster initialization, Argo CD installation, authentication, and dashboard access.

## Repository & Credential Evidence

<img src="images/argocd/argo 1/argo1.4.png" width="450"/>
<img src="images/argocd/argo 1/argo1.5.png" width="450"/>
<img src="images/argocd/argo 3/argo3.1.png" width="450"/>
<img src="images/argocd/argo 3/argo3.3.png" width="450"/>
<img src="images/argocd/argo 3/argo3.4.png" width="450"/>
<img src="images/argocd/argo 3/argo3.5.png" width="450"/>
<img src="images/argocd/argo 3/argo3.6.png" width="450"/>
<img src="images/argocd/argo 3/argo3.7.png" width="450"/>

Documents GitHub PAT configuration, Argo CD API authentication, and repository registration.

## Deployment Verification Evidence

<img src="images/argocd/argo 4/argo4.1.png" width="450"/>
<img src="images/argocd/argo 4/argo4.2.png" width="450"/>
<img src="images/argocd/argo 4/argo4.3.png" width="450"/>

Documents GHCR integration, Kubernetes secrets, RBAC, and live deployment verification.

## GitOps Application Evidence

<img src="images/argocd/argo 5/argo5.1.png" width="450"/>
<img src="images/argocd/argo 5/argo5.2.png" width="450"/>
<img src="images/argocd/argo 5/argo5.3.png" width="450"/>
<img src="images/argocd/argo 5/argo5.4.png" width="450"/>
<img src="images/argocd/argo 5/argo5.5.png" width="450"/>

Documents declarative application configuration, GHCR image identity, Kubernetes workloads, and application execution.

## Monitoring Infrastructure Evidence

<img src="images/argocd/argo 6/argo6.1.png" width="450"/>
<img src="images/argocd/argo 6/argo6.2.png" width="450"/>
<img src="images/argocd/argo 6/argo6.3.png" width="450"/>
<img src="images/argocd/argo 6/argo6.4.png" width="450"/>
<img src="images/argocd/argo 6/argo6.5.png" width="450"/>
<img src="images/argocd/argo 6/argo6.6.png" width="450"/>
<img src="images/argocd/argo 6/argo6.7.png" width="450"/>
<img src="images/argocd/argo 6/argo6.8.png" width="450"/>
<img src="images/argocd/argo 6/argo6.9.png" width="450"/>
<img src="images/argocd/argo 6/argo6.10.png" width="450"/>
<img src="images/argocd/argo 6/argo6.11.png" width="450"/>
<img src="images/argocd/argo 6/argo6.12.png" width="450"/>

Documents Prometheus, Grafana, Helm installation, monitoring data sources, and dashboard configuration.

## Application Observability Evidence

<img src="images/argocd/argo 7/argo7.1.png" width="450"/>
<img src="images/argocd/argo 7/argo7.2.png" width="450"/>
<img src="images/argocd/argo 7/argo7.3.png" width="450"/>
<img src="images/argocd/argo 7/argo7.4.png" width="450"/>
<img src="images/argocd/argo 7/argo7.5.png" width="450"/>
<img src="images/argocd/argo 7/argo7.6.png" width="450"/>
<img src="images/argocd/argo 7/argo7.7.png" width="450"/>

Documents application metrics, Prometheus target discovery, and Grafana infrastructure dashboards.

## Alerting & Notification Evidence

<img src="images/argocd/argo 8/argo8.4.png" width="450"/>
<img src="images/argocd/argo 8/argo8.5.png" width="450"/>
<img src="images/argocd/argo 8/argo8.6.png" width="450"/>
<img src="images/argocd/argo 8/argo8.7.png" width="450"/>
<img src="images/argocd/argo 8/argo8.8.png" width="450"/>


Documents Prometheus alert rules, zero-replica alert generation, Alertmanager configuration, runtime logs, and Gmail notifications.

---

# 📊 Monitoring & Alerting Architecture

The observability architecture separates metrics collection, visualization, alert evaluation, and notification delivery.

```text
              KUBERNETES CLUSTER
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
   APPLICATION METRICS      NODE METRICS
          │                       │
          ▼                       ▼
     /metrics                 Node Exporter
          │                       │
          └───────────┬───────────┘
                      │
                      ▼
                  PROMETHEUS
                      │
             ┌────────┴────────┐
             ▼                 ▼
        Grafana          Alert Rules
             │                 │
             ▼                 ▼
        Dashboards        Alertmanager
                               │
                               ▼
                         SMTP Receiver
                               │
                               ▼
                             Gmail
```

This design enables the platform to monitor application behavior, infrastructure health, resource utilization, and workload availability.

---

# 🛡️ GitOps Continuous Deployment & Supply-Chain Security Model

The final architecture extends the existing secure software delivery lifecycle into Kubernetes.

The production image is built and published through the upstream CI/CD pipeline, with immutable digest identification, cryptographic signing, and SBOM generation.

Argo CD then manages Kubernetes deployment through declarative GitOps configuration.

The resulting architecture is:

```text
SOURCE CODE
    │
    ▼
GITHUB REPOSITORY
    │
    ▼
DEVSECOPS VALIDATION
    │
    ├── Jest
    ├── ESLint
    ├── OWASP
    ├── SonarCloud
    ├── Trivy
    ├── Gitleaks
    ├── Cypress
    └── k6
    │
    ▼
PRODUCTION BUILD
    │
    ▼
MULTI-ARCHITECTURE IMAGE
    │
    ▼
GHCR + DOCKER HUB
    │
    ▼
IMMUTABLE SHA-256 DIGEST
    │
    ▼
COSIGN SIGNATURE
    │
    ▼
SPDX SBOM
    │
    ▼
GITOPS APPLICATION
    │
    ▼
ARGO CD
    │
    ▼
KUBERNETES RECONCILIATION
    │
    ▼
DEPLOYMENT VERIFICATION
    │
    ▼
RUNNING APPLICATION
    │
    ▼
PROMETHEUS METRICS
    │
    ▼
GRAFANA DASHBOARDS
    │
    ▼
PROMETHEUS ALERTING
    │
    ▼
ALERTMANAGER
    │
    ▼
EMAIL NOTIFICATION
```

This provides a continuous operational workflow connecting source control, secure artifact delivery, Kubernetes deployment, and observability.

---

# 🔑 Credential & Access Management

The implementation separates repository authentication, Argo CD API authentication, container registry access, and monitoring notification credentials.

The documented credential categories include:

```text
GitHub
├── Fine-Grained GitHub PAT
├── ARGOCD_AUTH_TOKEN
└── ARGOCD_SERVER

Argo CD
├── admin
└── pipeline-verifier

Container Registry
└── ghcr-secret

Kubernetes
├── Registry Authentication
├── Deployment Verifier Secrets
└── Service Account / RBAC

Monitoring
└── Alertmanager SMTP Credentials
```

The credentials are provisioned through the respective GitHub and Kubernetes secret-management mechanisms.

Secret values should not be committed to the source repository.

---

# 📁 Project Structure

The documented implementation uses the following project components:

```text
resume-matcher-devops/
│
├── argocd-app.yaml
├── argocd-rbac-setup.yaml
├── argocd-deployment-verifier.yaml
│
├── ghcr.sh
│
├── .github/
│   └── workflows/
│       └── GitHub Actions workflows
│
├── k8s/
│   └── manifests/
│       ├── Application manifests
│       ├── Deployment manifests
│       ├── Service manifests
│       ├── Monitoring manifests
│       └── Prometheus configuration
│
├── Kubernetes/
│   ├── Argo CD configuration
│   ├── RBAC configuration
│   ├── Deployment verification
│   └── Monitoring configuration
│
├── images/
│   └── argocd/
│       ├── argo 1/
│       ├── argo 2/
│       ├── argo 3/
│       ├── argo 4/
│       ├── argo 5/
│       ├── argo 6/
│       ├── argo 7/
│       └── argo 8/
│
└── application source
```

---

# 🚀 Execution Summary

## 1. Remote Infrastructure Initialization

Initialize zrok and establish remote connectivity.

```bash
zrok enable <zrok-token>

zrok.exe access private \
  --bind 127.0.0.1:9000 \
  <share-token>
```

Establish SSH port forwarding to the remote infrastructure.

## 2. Kubernetes & Argo CD Initialization

```bash
kind create cluster

kubectl cluster-info

kubectl create namespace argocd

kubectl apply --server-side \
  -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get pods -n argocd
```

## 3. Repository Integration

```bash
argocd repo add \
  https://github.com/kaifmohammedkhan/resume-matcher-devops.git \
  --username kaifmohammedkhan \
  --password <github-personal-access-token>

argocd repo list
```

## 4. GHCR & Deployment Verification

```bash
chmod +x ghcr.sh && ./ghcr.sh

kubectl apply -f argocd-rbac-setup.yaml

kubectl apply -f argocd-deployment-verifier.yaml

kubectl logs job/argocd-deployment-verifier-job \
  -f -n argocd
```

## 5. GitOps Application Deployment

Apply the declarative Argo CD application manifest.

```bash
kubectl apply -f argocd-app.yaml
```

Verify the deployed application through the Argo CD dashboard and Kubernetes resources.

## 6. Monitoring Installation

```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update

kubectl create namespace monitoring

helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring
```

## 7. Application Metrics Validation

```bash
curl http://localhost:3000/metrics
```

Verify Prometheus target discovery and Grafana dashboards.

## 8. Alerting Validation

```bash
kubectl scale deployment resume-matcher-ghcr \
  --replicas=0 \
  -n default
```

Inspect Prometheus alert states and Alertmanager logs.

Verify the resulting email notification through Gmail.

---

# 🎯 Project Outcome

The completed implementation demonstrates an integrated **Argo CD GitOps Continuous Deployment and Kubernetes Observability architecture** extending the Resume Matcher DevSecOps platform beyond CI/CD and container publishing.

The architecture connects secure artifact delivery with declarative Kubernetes deployment, automated synchronization, live deployment verification, infrastructure monitoring, application telemetry, alert evaluation, and email-based incident notification.

The final implementation demonstrates:

```text
                 GITOPS CONTINUOUS DELIVERY
                            │
                            ▼
                     Declarative State
                            │
                            ▼
                         Argo CD
                            │
                            ▼
                    Kubernetes Cluster
                            │
                            ▼
                   Automated Synchronization
                            │
                            ▼
                    Immutable Image Identity
                            │
                            ▼
                   Deployment Verification
                            │
                            ▼
                     Running Workloads
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
         Prometheus      Grafana      Alertmanager
             │              │              │
             ▼              ▼              ▼
          Metrics       Dashboards      Alert Rules
             │              │              │
             └──────────────┼──────────────┘
                            │
                            ▼
                      SMTP / Gmail
```

The result is a comprehensive **GitOps and Kubernetes observability platform** demonstrating the transition from secure container delivery to declarative deployment management and operational monitoring.

---

# 👤 Author

**Kaif Mohammed Khan**

DevOps / DevSecOps project demonstrating Argo CD GitOps continuous deployment, Kubernetes orchestration, declarative application management, GitHub integration, immutable container image verification, Prometheus monitoring, Grafana visualization, Alertmanager integration, and automated operational email notifications.

---

<div align="center">

**© 2026 Kaif. All rights reserved.**

</div>
