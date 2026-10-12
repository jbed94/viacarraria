# External Production Services Specification & Configuration Guide

This document defines all external cloud, networking, persistence, communication, and monetization services required to deploy and operate **Via Carraria** in a production environment.

Related configuration files:
- Helm Production Values: [`viacarraria-infrastructure/platforms/helm/graph-app/values-production.yaml`](file:///home/jbed94/Projects/viacarraria/viacarraria-infrastructure/platforms/helm/graph-app/values-production.yaml)
- Helm GCP Values: [`viacarraria-infrastructure/platforms/helm/graph-app/values-gcp.yaml`](file:///home/jbed94/Projects/viacarraria/viacarraria-infrastructure/platforms/helm/graph-app/values-gcp.yaml)
- Helm Architecture & Runbook: [`viacarraria-infrastructure/platforms/helm/README.md`](file:///home/jbed94/Projects/viacarraria/viacarraria-infrastructure/platforms/helm/README.md)

---

## 1. Architecture Overview

In local development and Minikube environments, stateful services (PostgreSQL, Redis, MinIO, RabbitMQ, Weaviate) run as containerized pods inside the cluster. In production, persistent and stateful workloads should be offloaded to managed external cloud providers for high availability, automatic failover, point-in-time recovery (PITR), and horizontal scalability.

```
                           [ Cloudflare / Route 53 DNS ]
                                         │
                                         ▼
                             [ Ingress NGINX / TLS ]
                        (Let's Encrypt via cert-manager)
                                   │         │
                     ┌─────────────┘         └──────────────┐
                     ▼                                      ▼
           viacarraria-frontend                    viacarraria-backend
        (React Flow Canvas Engine)                  (NestJS Core API)
                     │                                      │
                     │                 ┌────────────────────┼────────────────────┐
                     ▼                 ▼                    ▼                    ▼
             [ Google AdSense ] [ Managed Postgres ] [ Managed Redis ] [ S3 / GCS Storage ]
                                (AWS RDS / GCP SQL)  (ElastiCache)     (Documents & Notes)
                                       │                    │
                                       ▼                    ▼
                              [ Managed RabbitMQ ]   [ Transactional SMTP ]
                              (CloudAMQP / Cluster)  (Resend / SendGrid)
                                       │                    │
                                       ▼                    ▼
                             viacarraria-services    [ Stripe Billing ]
                             (FastStream + TEI +      (Webhook Gateway)
                              Docling Ingestion)
```

---

## 2. Core Infrastructure & Networking (Mandatory)

### 2.1 Domain Registrar & Authoritative DNS
- **Purpose**: Routes internet traffic to the cluster ingress controller / cloud load balancer.
- **Recommended Providers**: Cloudflare, AWS Route 53, Google Cloud Domains, OVH, Namecheap.
- **Required Records**:
  - `A` Record: `@` (or `viacarraria.com`) pointing to the external IP of the Cloud Load Balancer / NGINX Ingress controller.
  - `A` / `CNAME` Record: `api.viacarraria.com` pointing to the API gateway (if using subdomain routing).
  - `A` / `CNAME` Record: `admin.viacarraria.com` pointing to the Decoupled Admin Panel (or routed through zero-trust VPN).
  - `TXT` Records: SPF, DKIM, and DMARC verification records for transactional email delivery.

### 2.2 TLS / SSL Automated Certificates (`cert-manager` & Let's Encrypt)
- **Purpose**: Issues and automatically renews public TLS certificates for HTTPS termination.
- **Provider**: Let's Encrypt ACME Certificate Authority.
- **Configuration**:
  - ACME Server: `https://acme-v02.api.letsencrypt.org/directory`
  - Challenge Type: HTTP-01 (via Ingress HTTP solver) or DNS-01 (via Cloudflare / Route 53 API token).
  - Admin Contact Email: `security@viacarraria.com`.
  - Configured in Helm via [`ClusterIssuer`](file:///home/jbed94/Projects/viacarraria/viacarraria-infrastructure/platforms/helm/graph-app/values-production.yaml).

### 2.3 S3-Compatible Object Storage (Blob Store)
- **Purpose**: Persists raw source documents, syllabi, lecture notes, and PDF source files managed by [`StorageService`](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/src/common/services/storage.service.ts). Replaces in-cluster MinIO.
- **Recommended Providers**: AWS S3, Google Cloud Storage (GCS with S3 Interoperability HMAC keys), Cloudflare R2, Wasabi.
- **Required Configuration**:
  - Bucket Name: `viacarraria-production-sources`
  - Endpoint: `https://s3.<region>.amazonaws.com` (or `https://storage.googleapis.com` for GCS)
  - Region: e.g., `us-east-1` or `europe-west1`
  - Access Credentials: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- **Recommended Bucket Lifecycle Rules**:
  - Abort incomplete multipart uploads after 1 day.
  - Transition active objects to Infrequent Access (IA) after 90 days.
  - Archive cold documents to Glacier / Coldline after 365 days.

---

## 3. Communication & Identity Services

### 3.1 Transactional Email / SMTP Provider
- **Purpose**: Delivers account activation tokens, password reset links, and security alerts via [`MailService`](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/src/common/services/mail.service.ts).
- **Recommended Providers**: Resend, Postmark, SendGrid, AWS SES, Brevo.
- **Required Configuration**:
  - `SMTP_HOST`: e.g. `smtp.resend.com` / `smtp.sendgrid.net`
  - `SMTP_PORT`: `587` (STARTTLS) or `465` (TLS)
  - `SMTP_USER`: API username (e.g. `resend` or `apikey`)
  - `SMTP_PASS`: API token
  - `SMTP_FROM`: `Via Carraria <noreply@viacarraria.com>`
- **Email Authentication (Critical for Inbox Delivery)**:
  - **SPF**: `v=spf1 include:<provider-spf-domain> ~all`
  - **DKIM**: 2x CNAME keys provided by email dashboard.
  - **DMARC**: `v=DMARC1; p=reject; rua=mailto:dmarc-reports@viacarraria.com`

### 3.2 OAuth 2.0 Social Identity Providers (Google)
- **Purpose**: Enables passwordless social login via [`auth.ts`](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/src/auth.ts).
- **Provider**: Google Cloud Console (APIs & Services > Credentials).
- **Required Configuration**:
  - `GOOGLE_CLIENT_ID`: OAuth 2.0 client identifier.
  - `GOOGLE_CLIENT_SECRET`: OAuth 2.0 secret key.
  - Authorized JavaScript Origins: `https://viacarraria.com`
  - Authorized Redirect URIs: `https://viacarraria.com/api/auth/callback/google`

---

## 4. Managed Persistence & Queue Infrastructure

For production stability, replace local StatefulSets with managed cloud equivalents:

### 4.1 Managed Relational Database (PostgreSQL 16)
- **Purpose**: Primary system database for users, dynamic plans, graph definitions, JSONB node payloads, and audit logs.
- **Recommended Providers**: AWS RDS PostgreSQL (Aurora), GCP Cloud SQL for PostgreSQL, Supabase Enterprise.
- **Connection String**: `postgresql://<user>:<password>@<db-host>:5432/viacarraria?sslmode=require`
- **Production Settings**:
  - Storage: SSD with auto-expansion.
  - Backups: Automated daily snapshots with 14–30 day retention and Point-in-Time Recovery (PITR).
  - High Availability: Multi-AZ standby replica with automated failover.
  - Connection Pool: Use PgBouncer or Supavisor for pooling high-concurrency connections.

### 4.2 Managed In-Memory Cache (Redis 7)
- **Purpose**: Rate limiting ([`RateLimitModule`](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/src/common/modules/rate-limit.module.ts)), user session storage, token blacklist revocation, and temporary caching.
- **Recommended Providers**: AWS ElastiCache for Redis, GCP Memorystore for Redis, Upstash Redis (Serverless).
- **Configuration**:
  - `REDIS_URL`: `rediss://:<token>@<redis-host>:6379`
  - Eviction Policy: `volatile-lru` or `allkeys-lru`.
  - In-transit and at-rest TLS encryption enabled.

### 4.3 Managed Message Broker (RabbitMQ 3.13)
- **Purpose**: Ingestion task queue (`doc-ingest-tasks`) driving Python FastStream workers with KEDA scale-to-zero autoscaling.
- **Recommended Providers**: CloudAMQP (Managed RabbitMQ) or In-Cluster StatefulSet with Persistent Volume Claim (PVC).
- **Configuration**:
  - `RABBITMQ_URL`: `amqps://<user>:<password>@<amqp-host>/viacarraria`
  - High Availability queues (`x-ha-policy: all`) enabled for mission-critical ingestion queues.

---

## 5. Monetization & Security Services

### 5.1 Payment Processing & Subscription Webhooks (Stripe / LemonSqueezy)
- **Purpose**: Manages Pro/Enterprise recurring subscriptions, credit top-ups, and customer portal via [`BillingService`](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/src/billing/billing.service.ts).
- **Configuration**:
  - `STRIPE_SECRET_KEY`: `sk_live_...`
  - `STRIPE_WEBHOOK_SECRET`: `whsec_...`
  - Webhook Endpoint: `https://api.viacarraria.com/api/billing/webhook/stripe`
  - Subscribed Events: `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`, `invoice.payment_succeeded`.

### 5.2 Ad Monetization (Google AdSense)
- **Purpose**: Displays GDPR-compliant responsive ad banners to Free and Guest tier users, logging telemetry via [`AdsService`](file:///home/jbed94/Projects/viacarraria/viacarraria-backend/src/ads/ads.service.ts).
- **Configuration**:
  - `NEXT_PUBLIC_ADSENSE_CLIENT_ID`: `ca-pub-XXXXXXXXXXXXXXXX`
  - Ad Unit Slot IDs configured in AdSense dashboard.
  - Host root `ads.txt` file at `https://viacarraria.com/ads.txt`.

### 5.3 Zero-Trust Remote Access (Decoupled Admin Panel)
- **Purpose**: The Decoupled Admin Panel (`viacarraria-frontend/admin`) exposes sensitive user, billing, and system operations. It should not be exposed to the public internet.
- **Recommended Solutions**:
  - **Tailscale Kubernetes Operator**: Binds the admin service directly to a private Tailnet, accessible only by authenticated engineering/operations devices.
  - **Cloudflare Access (Zero Trust)**: Protects `admin.viacarraria.com` behind single sign-on (Google Workspace / GitHub / SAML).

---

## 6. Production Kubernetes Secrets Template

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: viacarraria-production-secrets
  namespace: viacarraria
type: Opaque
stringData:
  # Databases & Caching
  DATABASE_URL: "postgresql://app_user:prod_password@<cloud-sql-or-rds-host>:5432/viacarraria?sslmode=require"
  REDIS_URL: "rediss://:<token>@<managed-redis-host>:6379"
  RABBITMQ_URL: "amqps://<user>:<password>@<cloudamqp-host>/viacarraria"
  
  # Object Storage (AWS S3 or GCS)
  S3_ENDPOINT: "https://s3.us-east-1.amazonaws.com"
  S3_REGION: "us-east-1"
  S3_BUCKET: "viacarraria-production-sources"
  AWS_ACCESS_KEY_ID: "AKIAXXXXXXXXXXXXXXXX"
  AWS_SECRET_ACCESS_KEY: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"

  # Transactional Email (Resend / SendGrid / Postmark)
  SMTP_HOST: "smtp.resend.com"
  SMTP_PORT: "587"
  SMTP_USER: "resend"
  SMTP_PASS: "re_prod_xxxxxxxxxxxxxxxx"
  SMTP_FROM: "Via Carraria <noreply@viacarraria.com>"

  # Security & Authentication
  JWT_SECRET: "<64-byte-secure-hex-string>"
  ADMIN_JWT_SECRET: "<64-byte-secure-hex-string>"
  GOOGLE_CLIENT_ID: "xxxxxxxxxxxx.apps.googleusercontent.com"
  GOOGLE_CLIENT_SECRET: "GOCSPX-xxxxxxxxxxxxxxxxxxxx"

  # Monetization
  STRIPE_SECRET_KEY: "sk_live_xxxxxxxxxxxxxxxxxxxx"
  STRIPE_WEBHOOK_SECRET: "whsec_xxxxxxxxxxxxxxxxxxxx"
```
