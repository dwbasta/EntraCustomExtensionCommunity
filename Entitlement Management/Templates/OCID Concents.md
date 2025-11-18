# OCID Consents - Entitlement Management Custom Extension

This template demonstrates how to automatically request user consent when access packages are assigned using Entitlement Management custom extensions. The solution correlates access packages to specific ODIC consent requirements using an external data source.

---

## Overview

This solution uses:
- **Entitlement Management** to manage access package assignments
- **Custom Extension** to trigger consent request logic at specific stages
- **External Data Source** to correlate access packages to consent requirements
- **Logic App** to orchestrate the consent request workflow
- **Managed Identity** for secure, credential-free authentication

---

## Architecture

```
Access Package Request → Entitlement Management → Custom Extension → Logic App
                                                                          ↓
                                                                Managed Identity
                                                                          ↓
                                                              External Data Source
                                                                          ↓
                                                        Query Consent Requirements
                                                                          ↓
                                                          Request ODIC User Consent
                                                                          ↓
                                                        Log Consent Response
```

---

## Part 1: External Data Source for Consent Correlation

### Why Use an External Data Source?

Using an external data source to correlate access packages to consent requirements provides several advantages:

1. **Centralized Management**: Single source of truth for consent requirements
2. **Easy Updates**: Change consent requirements without modifying code
3. **Audit Trail**: Track changes to consent mappings over time
4. **Flexibility**: Support complex consent scenarios with multiple conditions
5. **Scalability**: Manage thousands of access packages efficiently

### Data Source Options

You can use any of the following data sources to store the consent correlation data:
1. Blob Tables -- what this Custom extention is going to use.
2. CSV file
3. MSSQL with On premise Data Gateway
4. Or you own homegrown solution

## Part 1: Creating a Custom Extension in Entitlement Management

### Step 1: Navigate to Entitlement Management

1. Open **Entra ID Admin Center** ([https://entra.microsoft.com](https://entra.microsoft.com))
2. Navigate to **Identity Governance** → **Entitlement Management**
3. in the left navigation, click **Catelogs** then select your catelog you want
4. In the left navigation, click **Custom extensions**
5. Click **Add custom extension**

### Step 2: Configure Basic Settings

1. **Name**: Enter `OCID-Consent-Extension`
2. **Description**: Enter `Requests user consent based on access package assignment`
3. **Entension Type**: Select **Request workflow**
4. **Extension Configuration**: Select **Launch and continue**
5. **Details** Create logic app Yes then Select your subscription, Resource group and logic app name `entitlement-consent-extension`
6. **Create a logic app** then review and create to finish

### Step 3: Switch to Code View for JSON Import

1. In the Logic App, click **Code view** (next to the Designer button)
2. You'll see the current workflow definition in JSON format
3. This is where you can import the complete workflow definition

### Step 4: Import Logic App JSON Workflow

#### Option A: Replace Entire Workflow Definition

1. In **Code view**, select all the existing JSON code (Ctrl+A)
2. Delete the existing code
3. Paste your complete Logic App JSON definition (see json code below)
4. Click **Save**

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
            "Get_Users_data": {
                "runAfter": {
                    "Set-APIURL": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{triggerBody()?['data']?['subject']?['userPrincipalName']}?$select=displayName,onPremisesExtensionAttributes,department,jobTitle",
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
            "Get_priv_ID_info": {
                "runAfter": {
                    "Manager_PARSE_EDIT": [
                        "Succeeded"
                    ]
                },
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/v1.0/users/@{body('User_Data_Parse')?['onPremisesExtensionAttributes']?['extensionAttribute15']}?$select=displayName,employeeId",
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
                            "value": "https://graph.microsoft.com/v1.0/servicePrincipals/efbf203f-08b0-4ad1-872c-cfe21cfb7580/synchronization/jobs/API2AAD.e0e1f74aa30042c6a65c917c4befb560.5de63f34-e2d0-442f-9f52-85c6a6b91d76/bulkUpload"
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
                    "uri": "@variables('APIURL')",
                    "method": "POST",
                    "headers": {
                        "Content-Type": "application/scim+json"
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
                    "uri": "https://graph.microsoft.com/v1.0/users/@{body('User_Data_Parse')?['onPremisesExtensionAttributes']?['extensionAttribute15']}/manager/$ref",
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


## Part 6: Enabling Managed Identity

### What is a Managed Identity?

A **Managed Identity** is an Azure feature that provides Azure resources with an automatically managed identity in Entra ID. It eliminates the need to store credentials.

Benefits:
- **No credential management**: Azure handles credential rotation
- **Secure**: Credentials never appear in code or configs
- **Integrated**: Works with Azure services and Microsoft Graph
- **Auditable**: All actions logged with managed identity

### Step 1: Enable System-Assigned Managed Identity

#### Using Azure Portal

1. Open your Logic App: `entitlement-consent-extension`
2. Navigate to **Settings** → **Identity**
3. Under **System assigned** tab:
   - Toggle **Status** to **On**
   - Click **Save**
   - Confirm by clicking **Yes**
4. Once enabled, an **Object (principal) ID** will be displayed

## Part 7: Assigning Permissions to Managed Identity

### Required Permissions

The managed identity needs different permissions based on your data source and consent requirements:

#### Core Permissions (All Scenarios):

1. **DelegatedPermissionGrant.ReadWrite.All**: Manage all delegated permission grants
2. **EntitlementManagement.Read.All**: To read access package details

#### powershell script

```

Install-Module Microsoft.Graph -Scope CurrentUser

Connect-MgGraph -Scopes "Application.Read.All","AppRoleAssignment.ReadWrite.All,RoleManagement.ReadWrite.Directory"
$graphApp = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

$PermissionName = "DelegatedPermissionGrant.ReadWrite.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'entitlement-consent-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

$PermissionName = "EntitlementManagement.Read.All"
$AppRole = $graphApp.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
$managedID = Get-MgServicePrincipal -Filter "DisplayName eq 'entitlement-consent-extension'"
New-MgServicePrincipalAppRoleAssignment -PrincipalId $managedID.Id -ServicePrincipalId $managedID.Id -ResourceId $graphApp.Id -AppRoleId $AppRole.Id

```

### Step 5: Grant Data Source Access

#### For Azure Storage Table:

```

# Grant Storage Table Data Reader role
$storageAccountName = "YOUR_STORAGE_ACCOUNT"
$resourceGroupName = "YOUR_RESOURCE_GROUP"
$managedIdentityObjectId = "YOUR_OBJECT_ID"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
New-AzRoleAssignment -ObjectId $managedIdentityObjectId `
    -RoleDefinitionName "Storage Table Data Reader" `
    -Scope $storageAccount.Id

Write-Host "Granted Storage Table Data Reader access" -ForegroundColor Green

```

## Part 8: Configuring Access Packages to Use Custom Extension

### Step 1: Navigate to Access Package

1. In **Entra ID Admin Center**, go to **Identity Governance** → **Entitlement Management**
2. Click **Access packages**
3. Select the access package you want to add consent requirements to

### Step 2: Edit Access Package Policy

1. Click on the policy you want to modify (or create a new one)
2. Scroll to **Custom extensions** section
3. Click **Add custom extension**

### Step 3: Configure Custom Extension for Policy

1. **Select custom extension**: Choose `OCID-Consent-Extension`
2. **Stage**: is **Assignment is granted** 
3. select another **Stage**: and select **Assignment is removed**
4. Click **Save**



### Step 4: Test the Configuration

1. Request the access package (as a test user or yourself)
2. After approval, the custom extension should trigger
3. Check Logic App run history for execution details
4. Verify consent request is sent to user by viewing the odic application and looking at the permissions
