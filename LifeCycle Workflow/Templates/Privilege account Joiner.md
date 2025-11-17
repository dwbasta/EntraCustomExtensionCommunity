# Privileged Account Joiner - Lifecycle Workflow Custom Extension

This template demonstrates how to automatically provision privileged admin accounts when users join the organization using Lifecycle Workflows custom extensions and the Inbound API Provisioning service.

---

## Overview

This solution uses:
- **Lifecycle Workflows** to detect when a user requires a privileged account
- **Custom Extension** to trigger the provisioning logic
- **Inbound API Provisioning Service** to create and manage admin accounts
- **Managed Identity** for secure, credential-free authentication

---

## Architecture

```
User Joins Organization → LCW Joiner Workflow → Custom Extension → Logic App
                                                                        ↓
                                                              Managed Identity
                                                                        ↓
                                                    Inbound API Provisioning Service
                                                                        ↓
                                                        Create Privileged Account
```

---

## Part 1: Inbound API Provisioning Service for Admin Accounts

### What is the Inbound API Provisioning Service?

The **Inbound API Provisioning Service** is a Microsoft Entra ID capability that allows you to programmatically provision users into your tenant through a standardized API. Instead of creating users directly via Microsoft Graph API, this service provides:

- **Bulk provisioning capabilities** with better performance
- **Attribute mapping and transformation** through provisioning configuration
- **Built-in deduplication and matching logic**
- **Audit logging and monitoring** through the provisioning logs
- **Schema validation** to ensure data consistency

### Why Use It for Admin Accounts?

Using the Inbound API Provisioning Service for privileged accounts offers several advantages:

1. **Centralized Management**: All admin account provisioning flows through a single, auditable service
2. **Attribute Mapping**: Define once how attributes should be mapped from source data to Entra ID
3. **Separation of Concerns**: Provisioning logic is separate from your custom extension logic
4. **Compliance**: Built-in logging meets audit requirements for privileged account creation
5. **Idempotency**: The service handles duplicate requests gracefully

### Setting Up the Inbound API Provisioning Application

#### Step 1: Create the API-Driven Provisioning Application

1. Navigate to **Entra ID Admin Center** → **Identity** → **Applications** → **Enterprise Applications**
2. Click **New application** → **Create your own application**
3. Select **Integrate any other application you don't find in the gallery (Non-gallery)**
4. Name it: `Admin Account Provisioning API`
5. Click **Create**

#### Step 2: Enable API-Driven Provisioning

1. In the newly created enterprise application, go to **Provisioning**
2. Click **Get started**
3. Set **Provisioning Mode** to **Automatic**
4. Under **Admin Credentials**, select **API-Driven**
5. Click **Test Connection** and **Save**

#### Step 3: Configure Attribute Mappings

1. In the **Provisioning** section, click **Edit attribute mappings**
2. Under **Mappings**, select **Provision Azure Active Directory Users**
3. Configure the following attribute mappings:

| Source Attribute | Target Attribute | Matching type |
|-----------------|------------------|-------------------|
| `userName` | `userPrincipalName` | Direct |


4. Click **Save** to apply the mappings

---

## Part 2: Creating a Blank Custom Extension for Lifecycle Workflows

### Step 1: Navigate to Lifecycle Workflows

1. Open **Entra ID Admin Center** ([https://entra.microsoft.com](https://entra.microsoft.com))
2. Navigate to **Identity Governance** → **Lifecycle Workflows**
3. In the left navigation, click **Custom Extensions**
4. Click **Add custom extension**

### Step 2: Configure Basic Settings

1. **Name**: Enter `Admin-Account-Joiner-Extension`
2. **Description**: Enter `Creates privileged admin accounts for new joiners`
3. Click **Next**

### Step 3: Configure Endpoint Details

1. **Endpoint configuration**: Select **Create new Logic App**
2. **Subscription**: Select your Azure subscription
3. **Resource Group**: Select existing or create new resource group
4. **Logic App Name**: Enter `lcw-admin-joiner-extension`
5. **Region**: Select your preferred Azure region
6. Click **Create Logic App**

> **Note**: This will create a new Consumption-based Logic App with a pre-configured HTTP trigger optimized for Lifecycle Workflow custom extensions.

### Step 4: Review and Create

1. Review the configuration summary:
   - Custom extension name
   - Target Logic App details
   - Authentication method (will be configured next)
2. Click **Create**

### Step 5: Verify Logic App Creation

1. Once created, navigate to **Azure Portal** → **Logic Apps**
2. Find your Logic App: `lcw-admin-joiner-extension`
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

#### Option A: Replace Entire Workflow Definition

1. In **Code view**, select all the existing JSON code (Ctrl+A)
2. Delete the existing code
3. Paste your complete the following code

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
                                            "workflowVersion": {
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
            "Condition": {
                "actions": {},
                "runAfter": {
                    "Manager_Assignment": [
                        "Succeeded"
                    ]
                },
                "else": {
                    "actions": {
                        "Delay": {
                            "type": "Wait",
                            "inputs": {
                                "interval": {
                                    "count": 5,
                                    "unit": "Minute"
                                }
                            }
                        },
                        "manager_Assignment2": {
                            "runAfter": {
                                "Delay": [
                                    "Succeeded"
                                ]
                            },
                            "type": "Http",
                            "inputs": {
                                "uri": "https://graph.microsoft.com/v1.0/users/@{variables('PRIV UPN')}/manager/$ref",
                                "method": "PUT",
                                "headers": {
                                    "Content-Type": "application/json"
                                },
                                "body": {
                                    "@@odata.id": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['manager']?['id']}"
                                },
                                "authentication": {
                                    "audience": "https://graph.microsoft.com/",
                                    "type": "ManagedServiceIdentity"
                                }
                            },
                            "runtimeConfiguration": {
                                "contentTransfer": {
                                    "transferMode": "Chunked"
                                }
                            }
                        }
                    }
                },
                "expression": {
                    "and": [
                        {
                            "equals": [
                                "@outputs('Manager_Assignment')?['statusCode']",
                                204
                            ]
                        }
                    ]
                },
                "type": "If"
            },
            "Delay_1": {
                "runAfter": {
                    "SCIM_api_Call_to_create_user": [
                        "Succeeded"
                    ]
                },
                "type": "Wait",
                "inputs": {
                    "interval": {
                        "count": 2,
                        "unit": "Minute"
                    }
                }
            },
            "Get_Users_Details": {
                "runAfter": {
                    "variables": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['userPrincipalName']}?$select=displayName,givenName,postalCode,mailnickname,jobTitle,surname,employeeId,department",
                    "method": "GET",
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Get_manager": {
                "runAfter": {
                    "Get_Users_Details": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['userPrincipalName']}?$expand=manager",
                    "method": "GET",
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Link_PRIV_Username_to_Users_Ext_15": {
                "runAfter": {
                    "set-privUpn": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['userPrincipalName']}",
                    "method": "PATCH",
                    "body": {
                        "onPremisesExtensionAttributes": {
                            "extensionAttribute15": "@{variables('PRIV UPN')}"
                        }
                    },
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Manager_Assignment": {
                "runAfter": {
                    "Delay_1": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{variables('PRIV UPN')}/manager/$ref",
                    "method": "PUT",
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "body": {
                        "@@odata.id": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['manager']?['id']}"
                    },
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Parse_ManagerINFO": {
                "runAfter": {
                    "Parse_User_details": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_manager')",
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
            "Parse_User_details": {
                "runAfter": {
                    "Get_manager": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_Users_Details')",
                    "schema": {
                        "properties": {
                            "department": {
                                "type": "string"
                            },
                            "displayName": {
                                "type": "string"
                            },
                            "employeeId": {
                                "type": "string"
                            },
                            "givenName": {
                                "type": "string"
                            },
                            "issuer": {
                                "type": "string"
                            },
                            "jobTitle": {
                                "type": "string"
                            },
                            "mailNickname": {
                                "type": "string"
                            },
                            "surname": {
                                "type": "string"
                            }
                        },
                        "type": "object"
                    }
                }
            },
            "SCIM_api_Call_to_create_user": {
                "runAfter": {
                    "Link_PRIV_Username_to_Users_Ext_15": [
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
                                "bulkId": "@{body('Parse_User_details')?['employeeId']}",
                                "data": {
                                    "displayName": "@{variables('AdminUPNNamingConvention')}@{body('Parse_User_details')?['displayName']}",
                                    "externalId": "@{variables('AdminEmployeeIDNamingConvention')}@{body('Parse_User_details')?['employeeId']}",
                                    "name": {
                                        "familyName": "@{body('Parse_User_details')?['surname']}",
                                        "givenName": "@{body('Parse_User_details')?['givenName']}"
                                    },
                                    "schemas": [
                                        "urn:ietf:params:scim:schemas:core:2.0:User",
                                        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
                                    ],
                                    "title": "@{body('Parse_User_details')?['jobTitle']}",
                                    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": {
                                        "department": "@{body('Parse_User_details')?['department']}",
                                        "emails": [
                                            {
                                                "value": "@{variables('PRIV UPN')}"
                                            }
                                        ]
                                    },
                                    "userName": "@{variables('PRIV UPN')}"
                                },
                                "method": "POST",
                                "path": "/Users"
                            }
                        ],
                        "failOnErrors": null,
                        "schemas": [
                            "urn:ietf:params:scim:api:messages:2.0:BulkRequest"
                        ]
                    },
                    "authentication": {
                        "audience": "https://graph.microsoft.com/",
                        "type": "ManagedServiceIdentity"
                    }
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "variables": {
                "runAfter": {},
                "type": "InitializeVariable",
                "inputs": {
                    "variables": [
                        {
                            "name": "AdminUPNNamingConvention",
                            "type": "string",
                            "value": "AZADM-"
                        },
                        {
                            "name": "AdminEmployeeIDNamingConvention",
                            "type": "string",
                            "value": "A"
                        },
                        {
                            "name": "APIURL",
                            "type": "string",
                            "value": "https://graph.microsoft.com/v1.0/servicePrincipals/SomeRandomnumber/synchronization/jobs/API2AAD.Aaaaaaaaaaaaaaaaaaa.11111111111111111/bulkUpload"
                        },
                        {
                            "name": "Onmicrosoftdomain",
                            "type": "string",
                            "value": "@@SOMEDOMAIN.onmicrosoft.com"
                        }
                    ]
                }
            },
            "set-privUpn": {
                "runAfter": {
                    "Parse_ManagerINFO": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable",
                "inputs": {
                    "variables": [
                        {
                            "name": "PRIV UPN",
                            "type": "string",
                            "value": "@{variables('AdminUPNNamingConvention')}@{body('Parse_User_details')?['mailNickname']}@{variables('Onmicrosoftdomain')}"
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

4. Click **Save**

### Step 8: Verify the Import

1. After saving, switch back to **Designer view**
2. Verify all actions and triggers are displayed correctly
3. Check for any warning or error icons on actions
4. Expand each action to confirm configuration is correct

### Step 9: Update the variable step
1.) Employee id naming convention for Admins
2.) Employee UPN naming convention for Admins
3.) Onmicrosoft Cloud UPN
4.) The API provisioning url found Under the API Enterprise app --> Provisioning --> Under Technical information --> Provisioning API Endpoint


> **Tip**: If you're importing a workflow from another environment, make sure to update any environment-specific values such as:
> - Connection references
> - Resource IDs
> - API endpoints
> - Managed identity references


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

1. Open your Logic App: `lcw-admin-joiner-extension`
2. Navigate to **Settings** → **Identity**
3. Under **System assigned** tab:
   - Toggle **Status** to **On**
   - Click **Save**
   - Confirm by clicking **Yes**
4. Once enabled, an **Object (principal) ID** will be displayed
5. **Copy this Object ID** - you'll need it for permission assignment

## Part 4: Assigning Permissions to Managed Identity

```
Install-Module Microsoft.Graph -Scope CurrentUser

Connect-MgGraph -Scopes "Application.Read.All","AppRoleAssignment.ReadWrite.All,RoleManagement.ReadWrite.Directory"
$graphApp = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

$PermissionName = "SynchronizationData-User.Upload"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-joiner-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

$PermissionName = "ProvisioningLog.Read.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-joiner-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

###For Manager Add
$PermissionName = "User.ReadWrite.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'lcw-admin-joiner-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

```

For this custom extension to work, the managed identity needs:

1. **SynchronizationData-User.Upload**: To send user data to the Inbound API Provisioning Service
2. **AuditLog.Read.All**: To read audit logs for verification and troubleshooting

