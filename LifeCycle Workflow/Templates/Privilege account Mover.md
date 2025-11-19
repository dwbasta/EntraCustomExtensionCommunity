# Privileged Account Mover - Lifecycle Workflow Custom Extension

This template demonstrates how to automatically update privileged admin accounts when users change roles, departments, or other attributes using Lifecycle Workflows custom extensions and the Inbound API Provisioning service.

---

## Overview

This solution uses:
- **Lifecycle Workflows** to detect when a user's attributes change
- **Custom Extension** to trigger the update logic
- **Inbound API Provisioning Service** to update admin account attributes
- **Managed Identity** for secure, credential-free authentication

---

## Architecture

```
User Attribute Changes → LCW Mover Workflow → Custom Extension → Logic App
                                                                      ↓
                                                            Managed Identity
                                                                      ↓
                                                  Inbound API Provisioning Service
                                                                      ↓
                                                    Update Privileged Account
```

---


### Why Use It for Admin Account Updates?

Using the Inbound API Provisioning Service for privileged account updates offers several advantages:

1. **Consistent Updates**: All admin account changes flow through a single, auditable service
2. **Attribute Transformation**: Apply business rules during attribute synchronization
3. **Correlation Management**: Reliably match user accounts to their privileged accounts
4. **Compliance**: Built-in logging meets audit requirements for privileged account modifications
5. **Idempotency**: The service handles duplicate update requests gracefully

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

1. **Name**: Enter `Admin-Account-Mover-Extension`
2. **Description**: Enter `Updates privileged admin accounts when user attributes change`
3. Click **Next**

### Step 3: Configure Endpoint Details

1. **Endpoint configuration**: Select **Create new Logic App**
2. **Subscription**: Select your Azure subscription
3. **Resource Group**: Select existing or create new resource group
4. **Logic App Name**: Enter `lcw-admin-mover-extension`
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
2. Find your Logic App: `lcw-admin-mover-extension`
3. Open the Logic App and click **Logic app designer**
4. You should see a blank HTTP trigger already configured with:
   - Trigger: **When a HTTP request is received**
   - Method: POST
   - Schema: Empty (will be defined based on actual payload)

### Step 6: Switch to Code View

1. In the Logic App, click **Code view** (next to the Designer button in the top toolbar)
2. You'll see the current workflow definition in JSON format
3. This is where you can import a pre-built workflow definition

### Step 7: Import Logic App JSON Workflow


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
            "Get_Manager": {
                "runAfter": {
                    "Get_Users_data": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    },
                    "method": "GET",
                    "uri": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['userPrincipalName']}?$expand=manager"
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Get_Users_data": {
                "runAfter": {
                    "Set-APIURL": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    },
                    "method": "GET",
                    "uri": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['userPrincipalName']}?$select=displayName,onPremisesExtensionAttributes,department,jobTitle"
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Get_priv_ID_info": {
                "runAfter": {
                    "Manager_PARSE_EDIT": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    },
                    "method": "GET",
                    "uri": "https://graph.microsoft.com/v1.0/users/@{body('User_Data_Parse')?['onPremisesExtensionAttributes']?['extensionAttribute15']}?$select=displayName,employeeId"
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Manager_PARSE_EDIT": {
                "runAfter": {
                    "User_Data_Parse": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_Manager')",
                    "schema": {
                        "properties": {
                            "givenName": {
                                "type": "string"
                            },
                            "manager": {
                                "properties": {
                                    "employeeId": {
                                        "type": "string"
                                    }
                                },
                                "type": "object"
                            },
                            "surname": {
                                "type": "string"
                            }
                        },
                        "type": "object"
                    }
                }
            },
            "Set-APIURL": {
                "runAfter": {},
                "type": "InitializeVariable",
                "inputs": {
                    "variables": [
                        {
                            "name": "APIURL",
                            "type": "string",
                            "value": "https://graph.microsoft.com/v1.0/servicePrincipals/randomnumbers/synchronization/jobs/API2AAD.aaaaaaaaaaaaaaaaaaaa.1111111111111111/bulkUpload"
                        }
                    ]
                }
            },
            "Update_Priv_SCIM_": {
                "runAfter": {
                    "parse_Priv_user": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    },
                    "body": {
                        "Operations": [
                            {
                                "bulkId": "@{body('User_Data_Parse')?['onPremisesExtensionAttributes']?['extensionAttribute15']}",
                                "data": {
                                    "displayName": "@{body('parse_Priv_user')?['displayName']}",
                                    "externalId": "@{body('parse_Priv_user')?['employeeId']}",
                                    "name": {
                                        "familyName": "@{body('Manager_PARSE_EDIT')?['surname']}",
                                        "givenName": "@{body('Manager_PARSE_EDIT')?['givenName']}"
                                    },
                                    "schemas": [
                                        "urn:ietf:params:scim:schemas:core:2.0:User",
                                        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
                                    ],
                                    "title": "@body('User_Data_Parse')?['jobTitle']"
                                },
                                "method": "POST",
                                "path": "/Users"
                            }
                        ],
                        "schemas": [
                            "urn:ietf:params:scim:api:messages:2.0:BulkRequest"
                        ]
                    },
                    "headers": {
                        "Content-Type": "application/scim+json"
                    },
                    "method": "POST",
                    "uri": "@variables('APIURL')"
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "User_Data_Parse": {
                "runAfter": {
                    "Get_Manager": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_Users_data')",
                    "schema": {
                        "properties": {
                            "department": {
                                "type": "string"
                            },
                            "jobTitle": {
                                "type": "string"
                            },
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
            "parse_Priv_user": {
                "runAfter": {
                    "Get_priv_ID_info": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_priv_ID_info')",
                    "schema": {
                        "properties": {
                            "displayName": {
                                "type": "string"
                            },
                            "employeeId": {
                                "type": "string"
                            }
                        },
                        "type": "object"
                    }
                }
            },
            "update_manager": {
                "runAfter": {
                    "Update_Priv_SCIM_": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    },
                    "body": {
                        "@@odata.id": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['manager']?['id']}"
                    },
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "method": "PUT",
                    "uri": "https://graph.microsoft.com/v1.0/users/@{body('User_Data_Parse')?['onPremisesExtensionAttributes']?['extensionAttribute15']}/manager/$ref"
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
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

> **Important**: Ensure your JSON is valid. The Logic App will validate the syntax when you save. If there are errors, they will be displayed at the bottom of the code view.

### Step 8: Verify the Import

1. After saving, switch back to **Designer view**
2. Verify all actions and triggers are displayed correctly
3. Check for any warning or error icons on actions
4. Expand each action to confirm configuration is correct

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

1. Open your Logic App: `lcw-admin-mover-extension`
2. Navigate to **Settings** → **Identity**
3. Under **System assigned** tab:
   - Toggle **Status** to **On**
   - Click **Save**
   - Confirm by clicking **Yes**
4. Once enabled, an **Object (principal) ID** will be displayed
5. **Copy this Object ID** - you'll need it for permission assignment

---

## Part 2: Assigning Permissions to Managed Identity

### Required Permissions

For this custom extension to work, the managed identity needs:

1. **SynchronizationData-User.Upload**: To send updated user data to the Inbound API Provisioning Service
2. **User.ReadWrite.All**: To read user attributes from the source user account
3. **AuditLog.Read.All**: To read audit logs for verification and troubleshooting

### PowerShell Script to Grant Permissions

```
 Install-Module Microsoft.Graph -Scope CurrentUser

Connect-MgGraph -Scopes "Application.Read.All","AppRoleAssignment.ReadWrite.All,RoleManagement.ReadWrite.Directory"
$graphApp = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

$PermissionName = "SynchronizationData-User.Upload"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-mover-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

$PermissionName = "ProvisioningLog.Read.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-mover-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

###For Manager Add and attribute read of user account
$PermissionName = "User.ReadWrite.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-mover-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

```

## Part 5: Creating the Mover Lifecycle Workflow

### Step 1: Navigate to Lifecycle Workflows

1. In **Entra ID Admin Center**, go to **Identity Governance** → **Lifecycle Workflows**
2. Click **Workflows**
3. Click **New workflow**

### Step 2: Configure Workflow Basics

1. **Category**: Select **Joiner, Leaver, Mover** or **Mover**
2. **Name**: Enter `Privileged Account Attribute Sync`
3. **Description**: Enter `Synchronizes attribute changes from user accounts to privileged admin accounts`
4. Click **Next**

### Step 3: Configure Execution Conditions

1. **Trigger type**: Select **Attribute changes**
2. **Attributes to monitor**:
   - `department`
   - `jobTitle`
   - `officeLocation`
   - `manager`
   - `employeeType`
3. **Scope**: Select specific groups or all users with privileged accounts
4. **Rule**: Add rule to only include users who have correlated admin accounts
   - Example: `extensionAttribute15 -eq "HasAdminAccount"`
5. Click **Next**

### Step 4: Configure Tasks

1. Click **Add task**
2. Under **Custom tasks**, select your custom extension: `Admin-Account-Mover-Extension`
3. **Task name**: Enter `Update Privileged Account Attributes`
4. **Description**: Enter `Calls Logic App to synchronize attributes to admin account`
5. Click **Add**
6. Click **Next**

### Step 5: Review and Create

1. Review the workflow configuration
2. **Enable workflow**: Toggle to **Yes** to activate immediately
3. Click **Create**

### Step 6: Test the Workflow

1. Select a test user who has a privileged account
2. Update one of the monitored attributes (e.g., change department)
3. Navigate to **Lifecycle Workflows** → **Workflows** → **Privileged Account Attribute Sync**
4. Click **Workflow history** to monitor execution
5. Verify the custom extension was called successfully
6. Check the privileged account to confirm attributes were updated

---

