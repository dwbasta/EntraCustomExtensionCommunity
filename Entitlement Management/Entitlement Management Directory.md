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

2. **Register the Extension**  
   - Use Microsoft Graph API to register a custom extension.  
   - Provide metadata such as `id`, `description`, and `target` (e.g., access package assignment).  

3. **Implement the Logic**  
   - Build an external service or API endpoint that the extension will call.  
   - Ensure it can handle authentication and authorization securely.  

4. **Bind to Entitlement Management**  
   - Attach the extension to an **access package policy** or **assignment workflow**.  
   - Configure triggers (e.g., on assignment, on removal).  

5. **Test in Non-Production**  
   - Validate the extension logic with test users and packages.  
   - Monitor Graph API calls and logs for errors.  

---

## 📑 Extension Templates

### 1️⃣ Governing Oath Consents

**Purpose:**  
In governance, consent is a critical control point. Custom extensions can enforce **Oath Consents** before granting entitlements, ensuring users explicitly acknowledge responsibilities or compliance requirements.

**Why Useful (Governance Approach):**  
- Provides an auditable record of user consent.  
- Ensures compliance with regulatory frameworks (e.g., GDPR, HIPAA, internal policies).  
- Adds a governance layer beyond simple access approval.  

**Example Workflow:**  
- User requests an access package.  
- Custom extension triggers an external consent form (e.g., digital signature service).  
- Access is only granted once consent is confirmed and logged.  

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
