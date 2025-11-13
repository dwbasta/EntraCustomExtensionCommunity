
# Custom Extensions in Microsoft Entra Lifecycle Workflows

## Overview
Custom extensions allow you to extend the capabilities of Microsoft Entra Lifecycle Workflows by integrating external systems or executing custom logic during identity lifecycle events. They are essential for scenarios where built-in tasks do not meet organizational requirements.

---

## How It Works
1. Create a **Lifecycle Workflow** for a specific event (Joiner, Mover, Leaver).
2. Add a **Custom Task Extension** to the workflow.
3. Configure the extension to call your API during execution.
4. Secure the integration with proper authentication and logging.

---
# LifeCycle Orcastration of Privilege Accounts.

## Security Considerations
- **High Privilege Access:** Custom extensions operate with elevated permissions. Apply least privilege principles. Use Managed Identities where possible and Restricted Access to the Custom Extentions.
> [!Warning]
> Custom Extention will need atleast Privilege Role Administrator if we are not modify the accounts directly. This can be avoided if you use privilege role groups and access packages.

- **Data Dependancy:** The way we get updated data for the privilege account is by refering to the user account that is correleated to the Privilege account. User account Updates first then Privilege account is update via customer extention to reflect changes from the user account. 
---

## 1.) Joiner setup


## 2.) Mover Setup


## 3.) Leaver Setup
