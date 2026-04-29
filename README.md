# azure_iam_lab
This project demonstrates how I designed and implemented a secure Azure access model for a multi-subscription cloud environment. The goal was to recreate a real-world enterprise pattern where a central security function can review, monitor and respond to workload environments without using permanent, overly broad permissions.  


In real cloud environments, production resources, logging, security operations and development workloads are usually separated into different accounts or subscriptions. This separation reduces risk, but it also requires a controlled way for security teams, deployment identities and incident responders to access the correct resources.

This lab shows that I understand how to:

- Design secure access across Azure subscriptions
- Apply least privilege with Azure RBAC
- Separate responsibilities between security, workload and logging environments
- Use Terraform to deploy repeatable cloud infrastructure
- Centralize activity logs for audit and detection
- Explain and document security design decisions clearly

---

## Business Scenario

A company has multiple Azure subscriptions:

The security team needs read-only visibility into workload resources, incident responders need limited emergency permissions, and deployment automation needs scoped access to deploy resources without receiving full subscription ownership.

---

## Architecture

```text
Microsoft Entra ID Tenant
│
├── Security Subscription
│   └── Security groups and privileged access model
│
├── Workload Subscription
│   ├── Azure RBAC assignments
│   ├── Custom incident response role
│   └── Deployment identity with scoped access
│
└── Logging Subscription
    └── Log Analytics Workspace for centralized monitoring
```

---

## Azure Services and Concepts Used

| Area                   | Azure Technology                                               |
|------------------------|----------------------------------------------------------------|
| Identity               | Microsoft Entra ID                                             |
| Access control         | Azure RBAC                                                     |
| Privileged access      | Microsoft Entra Privileged Identity Management concept         |
| Infrastructure as Code | Terraform                                                      |
| Monitoring             | Azure Activity Log                                             |
| Log collection         | Log Analytics Workspace                                        |
| Detection              | KQL queries                                                    |
| Governance             | Least privilege, scoped role assignments, separation of duties |

---

## What I Built

### 1. Security Groups for Access Separation

I designed separate Entra ID security groups for different operational responsibilities:

| Group                      | Purpose                                                  |
|----------------------------|----------------------------------------------------------|
| `GRP-AZ-Security-Audit`    | Read-only security review of workload resources          |
| `GRP-AZ-Incident-Response` | Limited emergency response permissions                   |
| `GRP-AZ-Deployment`        | Controlled deployment access for automation or operators |

This avoids assigning permissions directly to individual users and makes access easier to review, revoke and audit.

---

### 2. Read-Only Security Audit Access

The audit group was assigned read-only permissions in the workload subscription:

| Role            | Scope                 |
|-----------------|-----------------------|
| Reader          | Workload subscription |
| Security Reader | Workload subscription |

This allows security reviewers to inspect resources, configuration and security posture without being able to modify or delete infrastructure.

This demonstrates knowledge of:

- Azure RBAC role assignments
- Subscription-level scope
- Read-only security access
- Separation between visibility and administrative control

---

### 3. Custom Incident Response Role

I created a custom Azure RBAC role for incident response instead of using broad roles such as `Owner` or `Contributor`.

The role allows responders to read the environment and perform limited network containment actions, such as working with Network Security Groups.

Example permission areas:

```json
[
  "*/read",
  "Microsoft.Network/networkSecurityGroups/write",
  "Microsoft.Network/networkSecurityGroups/securityRules/read",
  "Microsoft.Network/networkSecurityGroups/securityRules/write",
  "Microsoft.Network/networkSecurityGroups/securityRules/delete"
]
```

This shows how to reduce blast radius during security incidents by giving responders only the permissions they need.

---

### 4. Scoped Deployment Identity

I created a deployment identity and assigned it access only to the workload resource group, not the full subscription.

| Identity            | Role                                  | Scope             |
|---------------------|---------------------------------------|-------------------|
| Deployment identity | Contributor or custom deployment role | `rg-workload-lab` |

In a production design, this should be replaced with a more restrictive custom deployment role and workload identity federation instead of long-lived secrets.

This demonstrates knowledge of:

- Automation identities
- Service principals or managed identities
- Scoped deployment access
- Avoiding unnecessary subscription-wide permissions

---

### 5. Centralized Logging

I created a Log Analytics Workspace in the logging subscription and configured the workload subscription to send Azure Activity Logs to it.

This provides a central place to investigate privileged activity and access changes.

Important events to monitor include:

- New role assignments
- Deleted role assignments
- Custom role definition changes
- Diagnostic setting changes
- Failed administrative operations
- Privileged access activation events

---

## Monitoring Queries

### Role Assignment Changes

```kql
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "Microsoft.Authorization/roleAssignments"
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue, SubscriptionId
| order by TimeGenerated desc
```

### Custom Role Definition Changes

```kql
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "Microsoft.Authorization/roleDefinitions"
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue, SubscriptionId
| order by TimeGenerated desc
```

### Diagnostic Setting Changes

```kql
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "diagnosticSettings"
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue, ResourceGroup, SubscriptionId
| order by TimeGenerated desc
```

### Failed Administrative Operations

```kql
AzureActivity
| where TimeGenerated > ago(24h)
| where CategoryValue == "Administrative"
| where ActivityStatusValue == "Failure"
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue, SubscriptionId
| order by TimeGenerated desc
```

---

## Terraform Implementation

The lab was implemented with Terraform to make the environment repeatable and easier to review.

Repository structure:

```
azure-cross-subscription-access-lab/
├── README.md
├── providers.tf
├── variables.tf
├── main.tf
├── outputs.tf
├── terraform.tfvars.example
└── docs/
    ├── design-decisions.md
    └── monitoring-kql.md
```

Terraform was used to manage:

- Azure resource groups
- Azure RBAC role assignments
- Custom RBAC role definition
- Deployment identity permissions
- Log Analytics workspace

--- 

## Validation Performed

- Audit group role assignments: `Reader` and `Security Reader` exist at workload subscription scope
- Incident response role: Custom role exists with limited permissions
- Deployment identity: Access is scoped to workload resource group
- Log Analytics workspace: Workspace exists in logging subscription
- Activity Log export: Workload subscription sends logs to Log Analytics
- KQL queries: Role assignment and administrative events are visible

---

## Skills Demonstrated

This project demonstrates practical knowledge in:

- Azure cloud security architecture
- Microsoft Entra ID access design
- Azure RBAC and least privilege
- Custom Azure role definitions
- Privileged access management concepts
- Terraform Infrastructure as Code
- Cross-subscription security design
- Centralized logging and monitoring
- KQL for security investigation
- Cloud governance and documentation

---

## Lessons Learned

This lab reinforced the importance of designing access around specific operational needs rather than giving broad administrative permissions.  
A secure cloud environment should separate identities, limit scopes, centralize logs and make privileged actions visible.

The most important takeaway is:
Secure cloud access is not only about giving the right people access. It is about giving the right people the minimum required access, at the right scope, with the right monitoring.

