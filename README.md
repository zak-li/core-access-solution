## Core Access Suite

> `Core Access Suite` is a unified, cloud-native Identity and Privileged Access Management (IAM/PAM) suite that centralizes enterprise secrets and enforces a strict Zero Trust security architecture. 

The platform seamlessly integrates public identity federation with restricted privileged access management, enabling secure authentication, authorization, and privileged operations across cloud-native environments. To mitigate credential sprawl and unauthorized privilege escalation, it replaces static credentials with short-lived, on-demand secrets and enforces granular Role-Based Access Control (RBAC).

## Table of Contents

- [Features](#features)
- [Quick Start](#quick-start)
- [Architecture](#architecture)
- [Observability](#observability)
- [Configuration](#configuration)
- [License](#license)

## Features

The production-grade infrastructure is provisioned through Infrastructure as Code (Terraform) and deployed on Azure Kubernetes Service (AKS). 

- **Identity Federation**: Authentication is strictly handled through `Auth0` using OAuth 2.0, OpenID Connect (OIDC), Single Sign-On (SSO), and mandatory Multi-Factor Authentication (MFA).
- **Dynamic Secrets**: `HashiCorp Vault` issues short-lived credentials on demand and automatically rotates them to mitigate lateral movement.
- **Service Mesh Encryption**: `Istio` secures all internal service-to-service communication with mutual TLS (mTLS), while a `Kong` API Gateway protects external endpoints.
- **Break-Glass Escrow**: `Azure Key Vault` provides a secure, automated escrow for Vault unseal keys and root tokens, ensuring safe recovery during outages.

**Built with:** Azure Kubernetes Service (AKS), HashiCorp Vault 1.17.6, Istio, Kong API Gateway, Auth0, Splunk 9.3, Angular 17.3, Terraform 1.5+, Azure Key Vault.

## Quick Start

You will need an authenticated `Azure CLI`, `Terraform` 1.5 or newer, and a configured `Auth0` Tenant.

### Setup

Clone the repository and prepare your environment variables:

```bash
git clone https://github.com/zak-li/core-access-suite.git
cd core-access-suite
cp terraform/terraform.tfvars.example terraform/terraform.tfvars
```

Export your Auth0 machine-to-machine credentials before provisioning:

```bash
export AUTH0_DOMAIN="dev-xxxx.eu.auth0.com"
export AUTH0_CLIENT_ID="<m2m_client_id>"
export AUTH0_CLIENT_SECRET="<m2m_secret>"
```

Execute the infrastructure deployment command to provision the base Azure infrastructure (VMs, AKS, Key Vault). Follow this with the app deployment command to install the Istio mesh, Kong, and the Angular application on Kubernetes:

```bash
make deploy-infra
make deploy-app
```

Alternatively, provision everything in one go:

```bash
make deploy
```

### Lifecycle Management

The `Makefile` exposes several commands to easily manage your environments without manually typing Terraform commands:

- `make stop` / `make start`: Suspend and resume Azure compute resources to save costs.
- `make unseal`: Unseal Vault automatically using the keys escrowed in Azure Key Vault.
- `make destroy`: Tear down the entire infrastructure (irreversible).
- `make fmt` / `make validate`: Format and validate your Terraform codebase.

## Architecture

`Auth0` serves as the identity provider for SSO and MFA. The public surface uses an `AKS` cluster equipped with a `Kong` API gateway and an `Istio` mesh. The privileged core runs on a hardened Azure virtual machine hosting `Vault` and `Splunk`. 

No direct path to the back-office tooling is exposed to the end user. When an operator requests privileged access, `Vault` redirects to `Auth0` for MFA validation. Once authenticated, the operator receives a token bound to specific RBAC policies.

For detailed request flows, refer to the [Architecture Document](docs/ARCHITECTURE.md).

## Observability

Every privileged action, API call, and secret generation is captured. `Splunk` runs continuously on the core VM, ingesting the `Vault` audit log along with the host logs in near real-time. This ensures total forensic observability and compliance with enterprise audit requirements.

## Configuration

The Terraform modules expect the following variables in your `terraform.tfvars` file or environment. 

| Variable | Description |
|---|---|
| `AUTH0_DOMAIN` | Auth0 tenant domain (e.g., `dev-xxx.eu.auth0.com`) |
| `AUTH0_CLIENT_ID` | Auth0 M2M application client ID |
| `AUTH0_CLIENT_SECRET` | Auth0 M2M application client secret |
| `location` | Azure region for deployment (default: `westeurope`) |
| `environment` | Environment tag for resources (e.g., `dev`, `prod`) |

*(See `terraform/terraform.tfvars.example` for the complete reference).*

## License

This project is released under the [MIT License](LICENSE).
