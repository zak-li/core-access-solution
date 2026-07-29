<br>

## Core Access Solution

> `Core Access Solution`: a unified, cloud-native Identity and Privileged Access Management (IAM/PAM) solution that centralizes enterprise secrets and enforces a strict Zero Trust security architecture. 

The platform seamlessly integrates public identity federation with restricted privileged access management, enabling secure authentication, authorization, and privileged operations across cloud-native environments. 

To mitigate credential sprawl and unauthorized privilege escalation, it replaces static credentials with short-lived, on-demand secrets and enforces granular Role-Based Access Control (RBAC).

## Table of Contents

- [Features](#features)
- [Quick Start](#quick-start)
- [Architecture](#architecture)
- [Observability](#observability)
- [Settings](#settings)
- [License](#license)

## Features

The core architecture bridges public identity federation with restricted infrastructure access. Authentication is enforced via `Auth0` (provisioned by [`terraform/modules/auth0`](terraform/modules/auth0/)) using strict MFA, OIDC, and SSO. Once authenticated, operators interact with `HashiCorp Vault` (configured via [`install.sh.tpl`](terraform/modules/compute/templates/install.sh.tpl)), which dynamically generates short-lived credentials and automatically rotates them. To ensure safe recovery, the system leverages `Azure Key Vault` ([`terraform/modules/key-vault`](terraform/modules/key-vault/)) as a secure, automated escrow for master unseal keys and root tokens.

The production-grade infrastructure is fully automated through Infrastructure as Code ([`Terraform`](terraform/)) and orchestrated on Azure Kubernetes Service (AKS). External API traffic is safely routed through a `Kong` gateway toward the [`apps/web`](apps/web/) Single Page Application. Internally, an `Istio` service mesh ([`kubernetes/`](kubernetes/)) encrypts all pod-to-pod communication with mutual TLS (mTLS), while all privileged actions are continuously audited and streamed to a centralized `Splunk` instance.

## Quick Start

The infrastructure requires the following command-line tools:

| Package | Version |
|---|---|
| `Azure CLI` | 2.50+ |
| `Terraform` | 1.5+ |
| `GNU Make` | 4.3+ |

The following components enable the full capabilities of the solution:

| Component | Role |
|---|---|
| `HashiCorp Vault` | Centralized secret management and dynamic credentials |
| `Auth0` | Identity federation (OAuth 2.0 / OIDC / MFA) |
| `Istio` | Zero Trust mutual TLS (mTLS) service mesh |
| `Kong API Gateway` | External API protection and routing |
| `Azure Kubernetes Service` | Scalable container orchestration |
| `Splunk` | Forensic audit logging and security SIEM |
| `Azure Key Vault` | Escrowed break-glass procedures |
| `Angular` | User-facing SSO dashboard |

### Setup

Clone from source and prepare your environment variables:

```bash
git clone https://github.com/zak-li/core-access-solution.git
cd core-access-solution
cp terraform/terraform.tfvars.example terraform/terraform.tfvars
```

The infrastructure runs entirely in Azure. Export your Auth0 machine-to-machine credentials before provisioning:

```bash
export AUTH0_DOMAIN="dev-xxxx.eu.auth0.com"
export AUTH0_CLIENT_ID="<m2m_client_id>"
export AUTH0_CLIENT_SECRET="<m2m_secret>"
```

Execute the infrastructure deployment using the [`Makefile`](Makefile) to provision the base Azure infrastructure (VMs, AKS, Key Vault):

```bash
make deploy-infra
```

Follow this with the app deployment command, executed via [`scripts/deploy-app.sh`](scripts/deploy-app.sh), to install the Istio mesh, Kong, and the Angular application on Kubernetes:

```bash
make deploy-app
```

### Lifecycle

The [`Makefile`](Makefile) exposes commands to easily manage your environments without manually typing Terraform commands:

```bash
make stop      # Suspend Azure compute resources to save costs
make start     # Resume Azure compute resources
make unseal    # Unseal Vault automatically using escrowed keys
make destroy   # Tear down the entire infrastructure (irreversible)
```

## Architecture

`Auth0` serves as the identity provider for SSO and MFA. The public surface uses an `AKS` cluster equipped with a `Kong` API gateway and an `Istio` mesh. The privileged core runs on a hardened Azure virtual machine hosting `Vault` and `Splunk`. 

No direct path to the back-office tooling is exposed to the end user. When an operator requests privileged access, `Vault` redirects to `Auth0` for MFA validation. Once authenticated, the operator receives a token bound to specific RBAC policies.

For detailed request flows and visual diagrams, refer to the [`Architecture Document`](docs/ARCHITECTURE.md).

## Observability

Every privileged action, API call, and secret generation is captured. `Splunk` runs continuously on the core VM alongside Vault, ingesting the `Vault` audit log and the host logs in near real-time. This ensures total forensic observability and compliance with enterprise audit requirements.

## Settings

The Terraform modules expect the following variables in your `terraform.tfvars` file or environment. The complete reference is in [`terraform/terraform.tfvars.example`](terraform/terraform.tfvars.example).

| Variable | Default | Description |
|---|---|---|
| `AUTH0_DOMAIN` | | Auth0 tenant domain (e.g., `dev-xxx.eu.auth0.com`) |
| `AUTH0_CLIENT_ID` | | Auth0 M2M application client ID |
| `AUTH0_CLIENT_SECRET` | | Auth0 M2M application client secret |
| `location` | `westeurope` | Azure region for deployment |
| `environment` | `dev` | Environment tag for resources |

## License

This project is licensed under the [`MIT License`](LICENSE).
