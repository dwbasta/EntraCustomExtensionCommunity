# 📂 Entitlement Management Custom Extensions Templates

This directory contains **template examples** for building custom extensions in Microsoft Entra Identity Governance, specifically focused on **Entitlement Management** scenarios.

⚠️ **Disclaimer**  
- These templates are provided **for educational purposes only**.  
- They are **not production-ready** and come **without warranty or support**.  
- Use at your own risk.  

---

## 🛠️ How to Create a New Custom Extension for Entitlement Management

1. **Define the Extension Purpose**  
   - Identify the business or governance requirement (e.g., external consent, distribution list management).  
   - Ensure it cannot be solved with native Entitlement Management features.  

2. **Create the Extension**  
   - Create the custom extension in a entitlement management catalog.  
   - Provide metadata such as `id`, `description`, and `target` (e.g., access package assignment).  

3. **Implement the Logic**  
   - Build an external service or API endpoint that the extension will call.  
   - Ensure it can handle authentication and authorization securely (preferably Managed Identity).  

4. **Bind to Entitlement Management**  
   - Attach the extension to an **access package policy** or **assignment workflow**.  
   - Configure triggers (e.g., on assignment, on removal).  

5. **Test in Non-Production**  
   - Validate the extension logic with test users and packages.  
   - Monitor Graph API calls and logs for errors.  

---

## 📑 Extension Templates

### 1️⃣ Governing OIDC Consents

> [!WARNING]
> **Advanced Custom Extension Template**  
> This is an advanced implementation requiring extensive knowledge of Azure Logic Apps, Microsoft Graph API, and Entra ID Identity Governance. It involves complex configuration including:
> - Azure Storage Tables setup and management
> - Managed Identity permissions configuration
> - Making Graph API calls to Entra
> - Advanced Logic App development
> 
> **Prerequisites:**
> - Strong understanding of Entra ID custom extensions
> - Experience with Azure Logic Apps and Microsoft Graph
> - Knowledge of OIDC/OAuth 2.0 protocols
> - Access to Azure subscription for storage and Logic Apps
> 
> This template is **not recommended for beginners**. Ensure you have proper testing environments before implementation.

**Purpose:**  
Automatically manage OIDC (OpenID Connect) user consents when access packages containing enterprise applications are assigned or removed. This custom extension correlates access packages to specific OIDC consent requirements using an external data source.

**Why Useful (Governance Approach):**  
- Provides an auditable record of user consent for application access
- Ensures compliance with regulatory frameworks (e.g., GDPR, HIPAA, internal policies)
- Automates the complex process of granting and revoking OIDC permissions
- Centrally manages consent requirements through external data correlation
- Adds a governance layer beyond simple access approval

**Example Workflow:**  
- User is granted an access package containing an enterprise application
- Custom extension reads the application's required OIDC permissions from Azure Storage Table
- Extension automatically grants the required delegated permissions to the user
- When access is revoked, the extension removes the OIDC consents

📄 **[OIDC Consents Template](./Templates/OIDC%20Consents.md)**  

---

### 2️⃣ Driving Distribution Lists (DL) Through Custom Extensions

**Purpose:**  
Organizations often want entitlement workflows to automatically update **Distribution Lists (DLs)** or Groups.  

**Why Not Possible with Access Packages Alone:**  
- Access Packages in Entitlement Management are designed to manage **resource roles** (apps, groups, SharePoint sites, etc.).  
- They do not natively support **direct Graph calls to Distribution Lists**.  
- DLs are legacy constructs in Exchange Online and are not fully manageable through the same Graph APIs used for modern groups.  

**Why Custom Extensions Help:**  
- A custom extension can call external logic that updates DL membership when an access package is assigned.  
- This bypasses the limitation of native Entitlement Management, while still keeping governance workflows intact.  

**Example Workflow:**  
- User is approved for an access package.  
- Custom extension triggers an API/service that adds the user to a Distribution List.  
- When access is revoked, the extension removes the user from the DL.  

---

## 🚀 Directory Layout
