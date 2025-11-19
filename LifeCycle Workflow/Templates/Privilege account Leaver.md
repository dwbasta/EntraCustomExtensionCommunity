# Privileged Account Leaver - Lifecycle Workflow Custom Extension

This template demonstrates how to automatically offboard privileged admin accounts when users leave the organization or no longer require elevated access using Lifecycle Workflows custom extensions and the Inbound API Provisioning service.

---

## Overview

This solution uses:
- **Lifecycle Workflows** to detect when a user leaves or loses privileged access
- **Custom Extension** to trigger the offboarding logic
- **Inbound API Provisioning Service** to disable and manage admin account lifecycle
- **Managed Identity** for secure, credential-free authentication

---

## Architecture

```
User Leaves Organization → LCW Leaver Workflow → Custom Extension → Logic App
                                                                        ↓
                                                              Managed Identity
                                                                        ↓
                                                    Inbound API Provisioning Service
                                                                        ↓
                                                      Disable Privileged Account
```

---


### Why Use API Provisioning Service for Admin Account Offboarding?

Using the Inbound API Provisioning Service for privileged account offboarding offers several advantages:

1. **Secure Deactivation**: Immediate account disablement through audited service
2. **Compliance Logging**: All offboarding actions are logged for audit requirements
3. **Retention Management**: Support for deferred deletion based on organizational policies
4. **Centralized Process**: All admin account offboarding flows through single service
5. **Rollback Safety**: Ability to track and potentially reverse actions if needed

### Setting Up the Inbound API Provisioning Application

>[!Note]
> Please refer to the 📄 **[Privileged Account Joiner Template](./Templates/Privilege%20account%20Joiner.md)** for inbound API provisioning setup 

>[!Important]
> The API provisioning will only work on the privilege accounts that are in an eligible state. If the active account has an assigned access on credentials check out, the custom extension will need higher privilege and cannot use the API provisioning service.


---

## Part 1: Creating a Blank Custom Extension for Lifecycle Workflows

### Step 1: Navigate to Lifecycle Workflows

1. Open **Entra ID Admin Center** ([https://entra.microsoft.com](https://entra.microsoft.com))
2. Navigate to **Identity Governance** → **Lifecycle Workflows**
3. In the left navigation, click **Custom Extensions**
4. Click **Add custom extension**

### Step 2: Configure Basic Settings

1. **Name**: Enter `Admin-Account-Leaver-Extension`
2. **Description**: Enter `Offboards privileged admin accounts for leavers`
3. Click **Next**

### Step 3: Configure Endpoint Details

1. **Endpoint configuration**: Select **Create new Logic App**
2. **Subscription**: Select your Azure subscription
3. **Resource Group**: Select existing or create new resource group
4. **Logic App Name**: Enter `lcw-admin-leaver-extension`
5. **Region**: Select your preferred Azure region
6. Click **Create Logic App**


### Step 4: Review and Create

1. Review the configuration summary:
   - Custom extension name
   - Target Logic App details
   - Authentication method (will be configured next)
2. Click **Create**

### Step 5: Verify Logic App Creation

1. Once created, navigate to **Azure Portal** → **Logic Apps**
2. Find your Logic App: `lcw-admin-leaver-extension`
3. Open the Logic App and click **Logic app designer**
4. You should see a blank HTTP trigger already configured with:
   - Trigger: **When a HTTP request is received**
   - Method: POST
   - Schema: Empty (will be defined based on actual payload)

### Step 6: Switch to Code View

1. In the Logic App, click **Code view** (next to the Designer button in the top toolbar)
2. You'll see the current workflow definition in JSON format
3. This is where you can import a pre-built workflow definition
4. Paste the following json into the Code view replacing the json

#### Sample JSON Structure

Your Logic App JSON should follow this structure:

```

{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "contentVersion": "1.0.0.0",
        "triggers": {
            "manual": {
                "type": "Request",
                "kind": "Http",
                "inputs": {
                    "schema": {
                        "properties": {
                            "data": {
                                "properties": {
                                    "callbackUriPath": {
                                        "description": "CallbackUriPath used for Resume Action",
                                        "title": "Data.CallbackUriPath",
                                        "type": "string"
                                    },
                                    "subject": {
                                        "properties": {
                                            "displayName": {
                                                "description": "DisplayName of the Subject",
                                                "title": "Subject.DisplayName",
                                                "type": "string"
                                            },
                                            "email": {
                                                "description": "Email of the Subject",
                                                "title": "Subject.Email",
                                                "type": "string"
                                            },
                                            "id": {
                                                "description": "Id of the Subject",
                                                "title": "Subject.Id",
                                                "type": "string"
                                            },
                                            "manager": {
                                                "properties": {
                                                    "displayName": {
                                                        "description": "DisplayName parameter for Manager",
                                                        "title": "Manager.DisplayName",
                                                        "type": "string"
                                                    },
                                                    "email": {
                                                        "description": "Mail parameter for Manager",
                                                        "title": "Manager.Mail",
                                                        "type": "string"
                                                    },
                                                    "id": {
                                                        "description": "Id parameter for Manager",
                                                        "title": "Manager.Id",
                                                        "type": "string"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "userPrincipalName": {
                                                "description": "UserPrincipalName of the Subject",
                                                "title": "Subject.UserPrincipalName",
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "task": {
                                        "properties": {
                                            "displayName": {
                                                "description": "DisplayName for Task Object",
                                                "title": "Task.DisplayName",
                                                "type": "string"
                                            },
                                            "id": {
                                                "description": "Id for Task Object",
                                                "title": "Task.Id",
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "taskProcessingResult": {
                                        "properties": {
                                            "createdDateTime": {
                                                "description": "CreatedDateTime for TaskProcessingResult Object",
                                                "title": "TaskProcessingResult.CreatedDateTime",
                                                "type": "string"
                                            },
                                            "id": {
                                                "description": "Id for TaskProcessingResult Object",
                                                "title": "TaskProcessingResult.Id",
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "workflow": {
                                        "properties": {
                                            "displayName": {
                                                "description": "DisplayName for Workflow Object",
                                                "title": "Workflow.DisplayName",
                                                "type": "string"
                                            },
                                            "id": {
                                                "description": "Id for Workflow Object",
                                                "title": "Workflow.Id",
                                                "type": "string"
                                            },
                                            "workflowVerson": {
                                                "description": "WorkflowVersion for Workflow Object",
                                                "title": "Workflow.WorkflowVersion",
                                                "type": "integer"
                                            }
                                        },
                                        "type": "object"
                                    }
                                },
                                "type": "object"
                            },
                            "source": {
                                "description": "Context in which an event happened",
                                "title": "Request.Source",
                                "type": "string"
                            },
                            "type": {
                                "description": "Value describing the type of event related to the originating occurrence.",
                                "title": "Request.Type",
                                "type": "string"
                            }
                        },
                        "type": "object"
                    }
                }
            }
        },
        "actions": {
            "Get_Linked_account": {
                "runAfter": {
                    "SET-APIURL": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['userPrincipalName']}?$select=displayName,onPremisesExtensionAttributes",
                    "method": "GET",
                    "authentication": {
                        "type": "ManagedServiceIdentity",
                        "audience": "https://graph.microsoft.com"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Parse_User_attributes": {
                "runAfter": {
                    "Get_Linked_account": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_Linked_account')",
                    "schema": {
                        "properties": {
                            "onPremisesExtensionAttributes": {
                                "properties": {
                                    "extensionAttribute15": {
                                        "type": "string"
                                    }
                                },
                                "type": "object"
                            }
                        },
                        "type": "object"
                    }
                }
            },
            "Get_Linked_Account_Details": {
                "runAfter": {
                    "Parse_User_attributes": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{body('Parse_User_attributes')?['onPremisesExtensionAttributes']?['extensionAttribute15']}?$select=displayname,employeeId",
                    "method": "GET",
                    "authentication": {
                        "type": "ManagedServiceIdentity",
                        "audience": "https://graph.microsoft.com"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Parse_PRIV_ID": {
                "runAfter": {
                    "Get_Linked_Account_Details": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_Linked_Account_Details')",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "@@odata.context": {
                                "type": "string"
                            },
                            "displayName": {
                                "type": "string"
                            },
                            "employeeId": {
                                "type": "string"
                            }
                        }
                    }
                }
            },
            "Disable_Priv_ID": {
                "runAfter": {
                    "Parse_PRIV_ID": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "@variables('APIURL')",
                    "method": "POST",
                    "headers": {
                        "Content-Type": "application/scim+json"
                    },
                    "body": {
                        "Operations": [
                            {
                                "bulkId": "@{body('Parse_PRIV_ID')?['employeeId']}",
                                "data": {
                                    "externalId": "@{body('Parse_PRIV_ID')?['employeeId']}",
                                    "schemas": [
                                        "urn:ietf:params:scim:schemas:core:2.0:User",
                                        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
                                    ],
                                    "active": false
                                },
                                "method": "POST",
                                "path": "/Users"
                            }
                        ],
                        "schemas": [
                            "urn:ietf:params:scim:api:messages:2.0:BulkRequest"
                        ]
                    },
                    "authentication": {
                        "type": "ManagedServiceIdentity",
                        "audience": "https://graph.microsoft.com"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "SET-APIURL": {
                "runAfter": {},
                "type": "InitializeVariable",
                "inputs": {
                    "variables": [
                        {
                            "name": "APIURL",
                            "type": "string",
                            "value": "https://graph.microsoft.com/v1.0/servicePrincipals/ramndomnumbers/synchronization/jobs/API2AAD.aaaaaaaaaaaaaaaaaaaaa.1111111111111111111111/bulkUpload"
                        }
                    ]
                }
            }
        },
        "outputs": {},
        "parameters": {
            "$connections": {
                "type": "Object",
                "defaultValue": {}
            }
        }
    },
    "parameters": {
        "$connections": {
            "type": "Object",
            "value": {}
        }
    }
}

```

### Step 8: Verify the Import

1. After saving, switch back to **Designer view**
2. Verify all actions and triggers are displayed correctly
3. Check for any warning or error icons on actions
4. Expand each action to confirm configuration is correct

#### Step 9: Update variables
1.) Click on the Set-APIURL step
2.) Put your api link in this section (found Under the API Enterprise app --> Provisioning --> Under Technical information --> Provisioning API Endpoint)

---

## Part 3: Enabling Managed Identity

### What is a Managed Identity?

A **Managed Identity** is an Azure feature that provides Azure resources with an automatically managed identity in Entra ID. It eliminates the need to store credentials in your code or configuration.

Benefits:
- **No credential management**: Azure handles credential rotation automatically
- **Secure**: Credentials never appear in code, configs, or logs
- **Integrated**: Works seamlessly with Azure resources and Microsoft Graph
- **Auditable**: All actions are logged with the managed identity as the actor

### Step 1: Enable System-Assigned Managed Identity on Logic App

#### Using Azure Portal

1. Open your Logic App: `lcw-admin-leaver-extension`
2. Navigate to **Settings** → **Identity**
3. Under **System assigned** tab:
   - Toggle **Status** to **On**
   - Click **Save**
   - Confirm by clicking **Yes**
4. Once enabled, an **Object (principal) ID** will be displayed
5. **Copy this Object ID** - you'll need it for permission assignment


## Part 4: Assigning Permissions to Managed Identity

### Required Permissions

For this custom extension to work, the managed identity needs:

1. **SynchronizationData-User.Upload**: To send deprovisioning data to the Inbound API Provisioning Service
2. **User.ReadWrite.All**: To read user attributes and verify account status
3. **AuditLog.Read.All**: To read audit logs for verification and compliance

### PowerShell Script to Grant Permissions

```

Install-Module Microsoft.Graph -Scope CurrentUser

Connect-MgGraph -Scopes "Application.Read.All","AppRoleAssignment.ReadWrite.All,RoleManagement.ReadWrite.Directory"
$graphApp = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

$PermissionName = "SynchronizationData-User.Upload"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-leaver-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

$PermissionName = "ProvisioningLog.Read.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-leaver-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

###For Manager Add and attribute read of user account
$PermissionName = "User.ReadWrite.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-leaver-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

```

## Part 5: Creating the Leaver Lifecycle Workflow

### Step 1: Navigate to Lifecycle Workflows

1. In **Entra ID Admin Center**, go to **Identity Governance** → **Lifecycle Workflows**
2. Click **Workflows**
3. Click **New workflow**

### Step 2: Configure Workflow Basics

1. **Category**: Select **Leaver**
2. **Name**: Enter `Privileged Account Offboarding`
3. **Description**: Enter `Disables and offboards privileged admin accounts when users leave`
4. Click **Next**

### Step 3: Configure Execution Conditions

#### Trigger Configuration

1. **Trigger type**: Select one of the following:
   - **Employee leaves the organization** (triggered by `employeeLeaveDateTime`)
   - **User account disabled** (triggered when `accountEnabled = false`)
   - **Group membership removed** (when removed from privileged access group)

2. **Timing**: 
   - For employee leave: Execute on or before leave date
   - For account disable: Execute immediately
   - For group removal: Execute within 1 hour

#### Scope Configuration

1. **Scope**: Select users with privileged accounts
2. **Rule expression**: Add rule to target only privileged account holders
   - Example: `extensionAttribute15 -eq "HasAdminAccount"`
   - Or: Member of group "Privileged-Account-Holders"
3. Click **Next**

### Step 4: Configure Tasks

#### Custom Extension Task

1. Click **Add task** → **Custom tasks**
2. Select your custom extension: `Admin-Account-Leaver-Extension`
3. **Task name**: Enter `Offboard Privileged Account`
4. **Description**: Enter `Disables and removes privileges from admin account`
5. **Run order**: After built-in tasks
6. Click **Add**
7. Click **Next**

### Step 5: Review and Create

1. Review the workflow configuration
2. **Enable workflow**: Toggle to **Yes** to activate immediately (or **No** for testing)
3. Click **Create**

### Step 6: Test the Workflow

#### Run On-Demand

1. Navigate to **Lifecycle Workflows** → **Workflows** → **Privileged Account Offboarding**
2. Click **Run on demand**
3. Select a test user
4. Click **Run workflow**
5. Monitor execution in **Workflow history**

