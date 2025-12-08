# 🔐 Unofficial Custom Extensions for Entra Identity Governance

Welcome to the **Unofficial Custom Extensions Repository** for Microsoft Entra Identity Governance.  
This repository provides **template examples** of custom extensions that can be used to extend Entra Identity Governance scenarios.  

⚠️ **Important Disclaimer**  
- These extensions are provided **as-is** for educational and template purposes only.  
- They are **not intended for production use**.  
- No warranty, guarantee, or official support is provided.  
- Use at your own risk.  

---

## 📘 What is a Custom Extension?

A **custom extension** in Entra Identity Governance is a way to integrate external logic or services into governance workflows.  
They allow you to:  
- Call external APIs or services during identity lifecycle events.  
- Add business-specific rules or automation beyond built-in capabilities.  
- Provide flexibility for unique organizational requirements.  

Think of them as "hooks" that let you extend Entra workflows with your own logic.

---

## 🔄 Using Custom Extensions in Lifecycle Workflows

Lifecycle workflows in Entra Identity Governance automate identity-related tasks such as onboarding, offboarding, and role changes.  
Custom extensions can be used to:  
- Trigger external processes when a user joins, leaves, or changes roles.  
- Notify third-party systems (e.g., HR, ITSM, or ticketing platforms).  
- Enforce organization-specific compliance checks.  
- Automate provisioning/deprovisioning in non-native systems.  

**Example Use Cases:**  
- Automatically send a welcome package request to HR when a new employee is onboarded.  
- Call an external API to disable accounts in a legacy system during offboarding.  
- Trigger a workflow to update a badge access system when a user changes departments.

**📖 [View Lifecycle Workflow Custom Extensions Guide](./LifeCycle%20Workflow/LifeCycle%20Workflow%20Directory.md)**  

---

## 🎟️ Using Custom Extensions in Entitlement Management

Entitlement Management in Entra Identity Governance helps manage access packages and approvals.  
Custom extensions can enhance this by:  
- Adding external approval logic (e.g., checking a compliance system before granting access).  
- Sending notifications to external systems when access is granted or revoked.  
- Logging entitlement decisions into third-party audit systems.  
- Enforcing additional business rules beyond built-in policies.  

**Example Use Cases:**  
- Require an external risk check before approving access to sensitive applications.  
- Notify a manager via a custom system when an access package is assigned.  
- Record entitlement decisions in a custom database for audit purposes.

**📖 [View Entitlement Management Custom Extensions Guide](./Entitlement%20Management/Entitlement%20Management%20Directory.md)**  

---

## 🚀 Getting Started

1. Browse the template examples in this repository.  
2. Adapt them to your organization's needs.  
3. Test thoroughly in **non-production environments**.  
4. Do not use these templates directly in production without proper validation.  

---

## 🤝 Contribute to This Repository

This repository is open for contributions!
Have an interesting custom extension or Logic App for governance scenarios? Follow these steps to contribute:

1. [Fork this Repo](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) - Click the Fork button at the top-right corner of this page.

2. Update your forked repo - Clone your forked repo locally and make your changes.

3. [Submit a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) - From your forked repo, create a pull request back to this repository.


## 📜 License & Support

- Provided **without warranty or support**.  
- Use for **educational and template purposes only**.  
- Contributions are welcome, but please note this is an **unofficial community resource**.  

---

### 🙌 Acknowledgement

This repository exists to help practitioners explore and experiment with extending **Microsoft Entra Identity Governance**.  
It is **not affiliated with or supported by Microsoft**.
