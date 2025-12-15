# Platform Administrator Guide

This guide helps **Platform Administrators** navigate ESO documentation based on your specific responsibilities.

## Who is this for?

You're a Platform Administrator if you:

- Deploy and administrate Kubernetes clusters
- Install and manage Kubernetes operators
- Configure cluster-wide resources and multi-tenant environments
- Ensure security, compliance, and operational standards
- Provide self-service tools to development teams

## Your Journey with ESO

### 1. Capabilties of ESO, providers available

**Goal:** Learn which functionalities are available.

| Feature | When to use | Documentation |
|---------|-------------|---------------|
| ExternalSecret | Sync secrets FROM external providers TO Kubernetes Secrets | [ExternalSecret API](api/externalsecret.md) |
| SecretStore / ClusterSecretStore | Configure connection to external secret providers | [SecretStore](api/secretstore.md), [ClusterSecretStore](api/clustersecretstore.md) |
| ExternalSecret | Sync multiple secrets FROM external providers TO a single Kubernetes Secret | [Templating Guide](guides/getallsecrets.md) |
| ClusterExternalSecret | Create ExternalSecrets across multiple namespaces from a single resource | [ClusterExternalSecret Guide](api/clusterexternalsecret.md) |
| PushSecret | Sync secrets FROM Kubernetes TO external providers (reverse sync) | [PushSecret Guide](guides/pushsecrets.md) |
| Generators | Generate passwords, keys, UUIDs, etc. instead of fetching from providers | [Generators](guides/generator.md) |
| Custom resources | Use ESO with ConfigMaps or other custom resources (not just Secrets) | [Targeting Custom Resources](guides/targeting-custom-resources.md) |
| Template transformations | Transform, merge, or template secret data before creating the Kubernetes Secret | [Templating Guide](guides/templating.md) |

**Goal:** learn how you can setup any provider (ie AWS Secrets Manager, Hashicorp Vault, etc)

| Task | Documentation |
|------|---------------|
| Create ClusterSecretStore | [ClusterSecretStore API](api/clustersecretstore.md) |
| AWS Secrets Manager integration | [AWS Secrets Manager](provider/aws-secrets-manager.md) |
| Azure Key Vault integration | [Azure Key Vault](provider/azure-key-vault.md) |
| Google Secret Manager integration | [Google Secret Manager](provider/google-secrets-manager.md) |
| HashiCorp Vault integration | [HashiCorp Vault](provider/hashicorp-vault.md) |
| All 40+ providers | [All 40+ providers](guides/introduction.md) |


**Key decisions:**
- **Shared ClusterSecretStore** - Centralized secret management, all teams use your ClusterSecretStore
- **Managed SecretStore per namespace** - You create isolated SecretStores per team
- **ESO as a Service** - Teams manage their own SecretStores, you provide the platform

---

### 2. Installation, Setup & Architectural Decisions

**Goal:** Get ESO installed and choose the right deployment model for your organization.

| What you need to do | Description | Documentation |
|---------------------|-------------|---------------|
| Install ESO with Helm | Deploy ESO operator in your Kubernetes cluster | [Getting Started](introduction/getting-started.md) |
| Understand ESO architecture and components | Learn how ESO works and what resources are available | [API Overview](introduction/overview.md) |
| Choose multi-tenancy model | Decide how teams will use ESO (shared, isolated, self-service) | [Multi-Tenancy Guide](guides/multi-tenancy.md) |
| Multiple controllers | Run separate ESO instances for different environments/teams | [Controller Classes](guides/controller-class.md) |
| Understand your options | Review deployment patterns and make architectural decisions | [Decision guide below](#quick-decision-guide) |

**Key decisions:**
- **Shared ClusterSecretStore** - Centralized secret management, all teams use your ClusterSecretStore
- **Managed SecretStore per namespace** - You create isolated SecretStores per team
- **ESO as a Service** - Teams manage their own SecretStores, you provide the platform

---

### 3. Security Configuration

**Goal:** Harden your ESO installation to meet security and compliance requirements.

| Security concern | Documentation |
|------------------|---------------|
| Complete security checklist | [Security Best Practices](guides/security-best-practices.md) |
| Namespace isolation | [Namespace-Scoped Installation](guides/security-best-practices.md#4-implement-namespace-scoped-installation) |
| Disable cluster-wide resources | [Disable Cluster Features](guides/disable-cluster-features.md) |
| Network traffic restrictions | [Network Security](guides/security-best-practices.md#network-traffic-and-security) |
| RBAC configuration | [RBAC Best Practices](guides/security-best-practices.md#role-based-access-control-rbac) |
| Verify container images | [Verify Artifacts](guides/security-best-practices.md#verify-artefacts) |
| Policy enforcement (Kyverno/OPA) | [Policy Engine Best Practices](guides/security-best-practices.md#policy-engine-best-practices) |
| Understand security model | [Threat Model](guides/threat-model.md) |

**Use:** ESO can be used for data exfiltration. Always implement NetworkPolicies and policy engines (such as OPA Gatekeeper, Kyverno, etc) to restrict unsafe usage.

---

### 4. Operations & Maintenance

**Goal:** Keep ESO healthy, monitor it, and troubleshoot issues.

| Operational task | Documentation |
|------------------|---------------|
| Monitor ESO with Prometheus | [API Overview - Monitoring](introduction/overview.md) (see metrics section) |
| Troubleshoot sync issues | [FAQ](introduction/faq.md) |
| Upgrade ESO | [Getting Started - Uninstalling](introduction/getting-started.md#uninstalling) + [Stability & Support](introduction/stability-support.md) |
| Regular security patches | [Security Best Practices - Regular Patches](guides/security-best-practices.md#regular-patches) |
| Use esoctl CLI tool | [Using esoctl Tool](guides/using-esoctl-tool.md) |

**Quick troubleshooting:**
```bash
# Check ExternalSecret status
kubectl describe externalsecret <name> -n <namespace>

# Check controller logs
kubectl logs -n external-secrets deployment/external-secrets -f

# Check SecretStore/ClusterSecretStore status
kubectl describe clustersecretstore <name>
```

---

### 5. Resources to Provide to Development Teams (FURther reading for development team)

**Goal:** Enable developers to use ESO effectively.

| What to provide | Documentation to share |
|-----------------|------------------------|
| Developer guide | [Application Developer Guide]() |
| How to create ExternalSecrets | [API - ExternalSecret](api/externalsecret.md) |
| Common patterns | [Guides](guides/introduction.md) |
| Examples | [Examples](examples/) |

---

## Quick Decision Guide

### Which deployment model should I use?

```
Do you have centralized secret management?
├─ Yes → Use ClusterSecretStore
│         │
│         ├─ All teams trust each other?
│         │  └─ Yes → Shared ClusterSecretStore (simplest)
│         │  └─ No → Add namespace selectors + policy engine
│         │
│         └─ Need strict isolation?
│            └─ Yes → Managed SecretStore per namespace
│
└─ No → Teams manage their own secrets?
         └─ Yes → ESO as a Service (self-service)
```

See: [Multi-Tenancy Guide](guides/multi-tenancy.md) for detailed comparisons.

### Should I use namespace-scoped or cluster-wide installation?

```
Do teams need to share ClusterSecretStores?
├─ Yes → Cluster-wide installation (default)
└─ No → Do you need strict isolation?
         ├─ Yes → Namespace-scoped installation
         └─ No → Cluster-wide but disable ClusterSecretStore CRD
```

See: [Security Best Practices - Namespace-Scoped Installation](guides/security-best-practices.md#4-implement-namespace-scoped-installation)

### Should I run multiple controllers?

```
Do you need different ESO configurations for different teams/environments?
├─ Yes → Use controller classes
└─ No → Single controller is sufficient
```

See: [Controller Classes](guides/controller-class.md)

---

## Security Checklist

Before going to production, ensure you've:

- [ ] Reviewed and hardened Helm chart values - [Harden Helm Chart](guides/security-best-practices.md#6-harden-the-helm-chart)
- [ ] Configured NetworkPolicies - [Network Security](guides/security-best-practices.md#network-traffic-and-security)
- [ ] Set up RBAC restrictions - [RBAC](guides/security-best-practices.md#role-based-access-control-rbac)
- [ ] Deployed policy engine (Kyverno/OPA) - [Policy Engine](guides/security-best-practices.md#policy-engine-best-practices)
- [ ] Disabled unused providers via policies
- [ ] Verified container images with cosign - [Verify Artifacts](guides/security-best-practices.md#verify-artefacts)
- [ ] Configured namespace selectors on ClusterSecretStores (if using) - [Match Conditions](guides/security-best-practices.md#2-configure-clustersecretstore-match-conditions)
- [ ] Disabled cluster-wide resources if not needed - [Disable Features](guides/security-best-practices.md#3-selectively-disable-reconciliation-of-cluster-wide-resources)
- [ ] Read the threat model - [Threat Model](guides/threat-model.md)

**Complete checklist:** [Security Best Practices](guides/security-best-practices.md)

---

## Common Scenarios

### Scenario: Centralized AWS Secrets Manager for all teams

**You want:** All teams to access AWS Secrets Manager through a shared ClusterSecretStore.

**What to do:**
1. Install ESO cluster-wide - [Getting Started](introduction/getting-started.md)
2. Create ClusterSecretStore with IAM role - [AWS Provider Guide](provider/aws-secrets-manager.md)
3. Add namespace selectors to restrict access - [Match Conditions](guides/security-best-practices.md#2-configure-clustersecretstore-match-conditions)
4. Deploy Kyverno policies to enforce secret prefixes - [Policy Engine](guides/security-best-practices.md#policy-engine-best-practices)
5. Configure NetworkPolicies - [Network Security](guides/security-best-practices.md#network-traffic-and-security)

### Scenario: Each team manages their own Vault namespace

**You want:** Teams to be autonomous with their own HashiCorp Vault namespaces.

**What to do:**
1. Install ESO cluster-wide - [Getting Started](introduction/getting-started.md)
2. Grant RBAC for teams to create SecretStores - [RBAC](guides/security-best-practices.md#role-based-access-control-rbac)
3. Provide SecretStore template for Vault - [Vault Provider Guide](provider/hashicorp-vault.md)
4. Deploy policies to restrict providers to Vault only - [Policy Engine](guides/security-best-practices.md#policy-engine-best-practices)
5. Share developer documentation - [Guides](guides/introduction.md)

See: [Multi-Tenancy Guide](guides/multi-tenancy.md) for more scenarios.

---

## Reference Documentation

### API Resources
- [ExternalSecret](api/externalsecret.md) - Primary resource developers use
- [SecretStore](api/secretstore.md) - Namespace-scoped secret provider configuration
- [ClusterSecretStore](api/clustersecretstore.md) - Cluster-wide secret provider configuration
- [ClusterExternalSecret](api/clusterexternalsecret.md) - Create ExternalSecrets across namespaces
- [PushSecret](api/pushsecret.md) - Push secrets TO external providers
- [Complete API Spec](api/spec.md)

### All Guides
- [Guides Index](guides/introduction.md) - Complete list of how-to guides

---

## Getting Help

- **Slack:** [#external-secrets on Kubernetes Slack](https://kubernetes.slack.com/messages/external-secrets)
- **Meetings:** [Bi-weekly development meetings](https://hackmd.io/GSGEpTVdRZCP6LDxV3FHJA) (Wednesdays 8PM Berlin Time)
- **Discussions:** [GitHub Discussions](https://github.com/external-secrets/external-secrets/discussions)
- **Issues:** [GitHub Issues](https://github.com/external-secrets/external-secrets/issues)

---

## Related Personas

Not a Platform Administrator? Check these guides:

- **Application Developer** - Using ESO to sync secrets in your applications → [App Developer Guide]()
- **Security Engineer** - Security auditing, compliance, rotation → [Security Guide]()
- **New to ESO** - Just getting started → [Getting Started](introduction/getting-started.md)
