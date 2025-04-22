# SETUP CLOUD ȘI DEVOPS

## 4.1 Infrastructură Terraform

**Structura repository Terraform:**
```
/
├── modules/
│   ├── gke/                 # Google Kubernetes Engine module
│   ├── cloudsql/            # Cloud SQL module
│   ├── apigee/              # Apigee configuration
│   ├── firebase/            # Firebase setup
│   ├── storage/             # Cloud Storage buckets
│   ├── monitoring/          # Monitoring setup
│   └── networking/          # VPC, firewall rules
├── environments/
│   ├── dev/                 # Development environment
│   ├── staging/             # Staging environment
│   └── prod/                # Production environment
├── main.tf                  # Main configuration
├── variables.tf             # Variable definitions
└── terraform.tfvars.example # Variable values template
```

## 4.2 Configurare Kubernetes și Apigee

**GKE Namespace Structure:**
- default
- kube-system
- schoolbus-system     # System services
- schoolbus-api        # API microservices
- schoolbus-data       # Data processing services
- schoolbus-realtime   # Real-time services
- monitoring           # Prometheus, Grafana
- cert-manager         # Certificate management

**Apigee Configuration:**
- API Products pentru grupare logică
- Developer portals pentru documentație
- Environment-specific deployments
- OAuth2 flows pentru autentificare

## 4.3 CI/CD Pipeline

**GitHub Actions Workflow pentru NestJS:**
```yaml
name: Build & Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build Docker image
        run: |
          docker build -t gcr.io/$PROJECT_ID/$SERVICE_NAME:$GITHUB_SHA .
          docker push gcr.io/$PROJECT_ID/$SERVICE_NAME:$GITHUB_SHA

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop'
    steps:
      - uses: actions/checkout@v2
      - name: Setup GCloud
        uses: google-github-actions/setup-gcloud@v0
      - name: Deploy to GKE
        run: |
          gcloud container clusters get-credentials $CLUSTER_NAME --zone $ZONE --project $PROJECT_ID
          kubectl apply -f k8s/deployment.yaml
          kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=gcr.io/$PROJECT_ID/$SERVICE_NAME:$GITHUB_SHA
```

## 4.4 Monitoring și Observabilitate

**Stack de Monitoring:**
- **Metrics**: Cloud Monitoring + Prometheus + Grafana
- **Logging**: Cloud Logging + Elasticsearch + Kibana
- **Tracing**: Cloud Trace + OpenTelemetry
- **Alerting**: Cloud Monitoring alerts + PagerDuty