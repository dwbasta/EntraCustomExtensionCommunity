
# Custom Extensions in Microsoft Entra Lifecycle Workflows

## Overview
Custom extensions allow you to extend the capabilities of Microsoft Entra Lifecycle Workflows by integrating external systems or executing custom logic during identity lifecycle events. They are essential for scenarios where built-in tasks do not meet organizational requirements.

Custom extensions enable organizations to:
- Integrate with external HR systems, ticketing systems, or provisioning platforms
- Execute complex business logic that cannot be achieved with built-in tasks
- Synchronize identity data across multiple systems
- Implement custom approval workflows
- Trigger notifications to external stakeholders
- Manage privileged accounts with specialized lifecycle requirements

---

## How It Works

### Architecture Overview
Custom extensions use an Azure Logic App to bridge Microsoft Entra ID with your custom endpoints. The workflow operates as follows:

1. **Create a Lifecycle Workflow** for a specific event (Joiner, Mover, Leaver)
2. **Add a Custom Task Extension** to the workflow
3. **Configure the Logic App** to receive webhook calls from Entra ID
4. **Implement your custom logic** to process the lifecycle event
5. **Return a response** to Entra ID indicating success or failure
6. **Secure the integration** with proper authentication and logging

### Technical Flow
```
User Event Trigger → Lifecycle Workflow → Custom Extension → Logic App → Custom API
                                                                    ↓
                                                              Process Logic
                                                                    ↓
                                                    Response ← Return Status
```

### Key Components
- **Custom Task Extension**: The configuration in Entra ID that defines when to call your extension
- **Logic App**: Azure resource that hosts the webhook endpoint and orchestrates the custom logic
- **Authentication**: OAuth 2.0 token validation to secure the communication
- **Callback**: Optional mechanism to return data back to Entra ID after processing

---

## Prerequisites

### Required Permissions
- **Lifecycle Workflows Administrator** or **Global Administrator** to create workflows
- **Application Administrator** to register the Logic App application
- **Azure Contributor** role to deploy Logic Apps and related resources

### Azure Resources Needed
- **Azure Subscription** with access to create Logic Apps
- **Azure Key Vault** (recommended) for storing secrets
- **Managed Identity** (optional but recommended) for secure authentication
- **Application Insights** (optional) for monitoring and diagnostics

---

# LifeCycle Orchestration of Privileged Accounts

## Use Case Overview
Managing privileged accounts (admin accounts, service accounts) requires special handling compared to regular user accounts. This section covers how to use custom extensions to orchestrate the lifecycle of privileged accounts based on their associated user accounts.

### Common Scenarios
- **Joiner**: Provision a privileged account when a user joins and requires administrative access
- **Mover**: Update privileged account attributes when the associated user changes roles or departments
- **Leaver**: Disable or delete privileged accounts when users leave or no longer require elevated access

---

## Security Considerations

### High Privilege Access
Custom extensions operate with elevated permissions. Apply least privilege principles:

- **Use Managed Identities**: Eliminate the need to store credentials by using Azure Managed Identities
- **Restrict Access**: Limit who can create, modify, or execute custom extensions
- **Audit Logging**: Enable comprehensive logging to track all extension executions
- **Conditional Access**: Apply CA policies to the service principals used by extensions
- **Network Security**: Use private endpoints or service endpoints where possible

> [!WARNING]
> Custom Extensions will need at least **Privileged Role Administrator** permission if directly modifying privileged accounts. This elevated permission requirement can be avoided by using privileged role groups and access packages instead of direct role assignments.

### Alternative Approach: Role Groups + Access Packages
Instead of granting Privileged Role Administrator to the extension:
1. Create Azure AD security groups for each privileged role
2. Assign Entra ID roles to these groups
3. Create Access Packages that grant membership to these groups
4. Use custom extensions to request/modify Access Package assignments (requires lower permissions)

### Data Dependency Model
The privileged account update pattern relies on a correlation between user and privileged accounts:

- **Primary Source of Truth**: The standard user account contains the authoritative data (department, location, manager, etc.)
- **Update Sequence**: User account is updated first (either manually or via HR sync)
- **Extension Trigger**: Lifecycle workflow detects the user account change
- **Privileged Account Sync**: Custom extension reads user account data and updates the correlated privileged account
- **Correlation Method**: Use extension attributes (e.g., `extensionAttribute1`) or naming conventions to link accounts

**Example Correlation Pattern:**
- User Account: `john.doe@contoso.com` with `employeeId = "12345"`
- Privileged Account: `john.doe-admin@contoso.com` with `extensionAttribute1 = "12345"`

---

## 1.) Joiner Setup


## 2.) Mover Setup


## 3.) Leaver Setup
