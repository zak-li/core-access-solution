# Core Access Suite

Built a unified, cloud-native Identity and Privileged Access Management (IAM/PAM) suite that centralizes enterprise secrets and enforces a Zero Trust security architecture. The platform seamlessly integrates identity federation with privileged access management, enabling secure authentication, authorization, and privileged operations across cloud-native environments. To mitigate credential sprawl and unauthorized privilege escalation, it replaces static credentials with short-lived, on-demand secrets and enforces granular Role-Based Access Control (RBAC).

The production-grade infrastructure is provisioned through Infrastructure as Code (Terraform) and deployed on Azure Kubernetes Service (AKS). It leverages an Istio service mesh to secure service-to-service communication with mutual TLS (mTLS), while Kong API Gateway protects external APIs. Authentication and identity federation are handled through Auth0 using OAuth 2.0, OpenID Connect (OIDC), Single Sign-On (SSO), and Multi-Factor Authentication (MFA). HashiCorp Vault and Azure Key Vault provide centralized secret management, automated credential rotation, dynamic secrets, and escrowed break-glass procedures, while Splunk delivers centralized audit logging and security monitoring.

**Built with:** Azure Kubernetes Service (AKS), HashiCorp Vault 1.17.6, Istio, Kong API Gateway, Auth0 (OAuth 2.0 / OIDC), Splunk 9.3, Angular 17.3, Terraform 1.5+, Azure Key Vault.

## Key Capabilities

`Vault` issues short lived credentials on demand to manage dynamic secrets. `Auth0` manages identity federation by enforcing mandatory `MFA` on every login. `Splunk` runs continuous audits by capturing privileged API calls for real time observability. 

- `Istio` provides complete encryption to secure internal communication with `mutual TLS`. 
- `Terraform` manages the automated, reproducible infrastructure as code.

## Architecture

`Auth0` serves as the identity provider for `SSO` and `MFA`. The public surface uses an `AKS` cluster equipped with a `Kong` API gateway and an `Istio` mesh. The privileged core runs on a hardened VM hosting `Vault` and `Splunk`. `Azure Key Vault` provides a secure escrow for root tokens and unseal keys. 

For detailed diagrams and request flows, refer to the [Architecture Document](docs/ARCHITECTURE.md) and view the [Architecture Diagram](.github/assets/architecture.png). You can explore the application in the [Web App Folder](apps/web/) and infrastructure in the [Terraform Directory](terraform/).

## Quick Start

You will need an authenticated `Azure CLI`, `Terraform` v1.5 or newer, and a configured `Auth0` Tenant. Start by copying the example variables file to `terraform.tfvars` inside the [Terraform Directory](terraform/). 

Configure your variables:
```bash
cp terraform/terraform.tfvars.example terraform/terraform.tfvars
export AUTH0_DOMAIN="dev-xxxx.eu.auth0.com"
export AUTH0_CLIENT_ID="<m2m_client_id>"
export AUTH0_CLIENT_SECRET="<m2m_secret>"
```

Execute the infrastructure deployment command to provision the base infrastructure. Follow this with the app deployment command to install `Istio`, `Kong`, and the application on `AKS`.

```bash
make deploy-infra
make deploy-app
```

## Project Commands

The [Makefile](Makefile) exposes several commands to manage the lifecycle of this project.

### Deployment

Provision everything:
```bash
make deploy
```

Provision base infrastructure, then app:
```bash
make deploy-infra
make deploy-app
```

### Lifecycle Management

Suspend and resume compute resources:
```bash
make stop
make start
```

Unseal `Vault` and destroy infrastructure (irreversible):
```bash
make unseal
make destroy
```

### Developer Tools

Format and validate `Terraform` code:
```bash
make fmt
make validate
```

Check shell scripts and view help:
```bash
make lint
make help
```

## License

This project is released under the [MIT License](LICENSE).
