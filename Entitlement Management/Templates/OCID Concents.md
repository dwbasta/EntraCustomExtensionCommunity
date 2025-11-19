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
3. MSSQL with Onpremise Data Gateway
4. Or you own homegrown solution

### Creating the Azure Tbale
```
# Variables#
$resourceGroup = MyResourceGroupn"
$location      = "EastUS"
$storageName   mystoragesta$(Get-Random)"   # must be globally unique and all lower case and numbers
$tableName     = "odiclookupGovernance"
$tenantid   "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"   # <-- your Tenant ID here


# Login to Azure
Connect-AzAccount -Tenant $tenantId

# Create Resource Group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create Storage Account
New-AzStorageAccount `
    -ResourceGroupName $resourceGroup `
    -Name $storageName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2

# Get Storage Account context
$ctx = New-AzStorageContext -StorageAccountName $storageName -UseConnectedAccount

# Create Table
New-AzStorageTable -Name $tableName -Context $ctx

Write-Host "Storage account '$storageName' created P## Turn off Application concent

with table '$tableName'"ame' in tenant '$tenantId'"ed with table '$tableName'"

````


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


### Step 3: Import Logic App JSON Workflow

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
                        "type": "object",
                        "properties": {
                            "AccessPackageAssignmentRequestId": {
                                "type": "string"
                            },
                            "CallbackUriPath": {
                                "type": "string"
                            },
                            "CustomExtensionStageInstanceId": {
                                "type": "string"
                            },
                            "Stage": {
                                "type": "string"
                            },
                            "RequestType": {
                                "type": "string"
                            },
                            "Answers": {
                                "type": "array"
                            },
                            "State": {
                                "type": "string"
                            },
                            "Status": {
                                "type": "string"
                            },
                            "CallbackConfiguration": {
                                "type": "object",
                                "properties": {
                                    "DurationBeforeTimeout": {
                                        "type": "string"
                                    }
                                }
                            },
                            "AccessPackage": {
                                "type": "object",
                                "properties": {
                                    "Id": {
                                        "type": "string",
                                        "description": "AccessPackage-Id"
                                    },
                                    "DisplayName": {
                                        "type": "string",
                                        "description": "AccessPackage-DisplayName"
                                    },
                                    "Description": {
                                        "type": "string",
                                        "description": "AccessPackage-Description"
                                    }
                                }
                            },
                            "AccessPackageCatalog": {
                                "type": "object",
                                "properties": {
                                    "Id": {
                                        "type": "string",
                                        "description": "AccessPackageCatalog-Id"
                                    },
                                    "DisplayName": {
                                        "type": "string",
                                        "description": "AccessPackageCatalog-DisplayName"
                                    },
                                    "Description": {
                                        "type": "string",
                                        "description": "AccessPackageCatalog-Description"
                                    }
                                }
                            },
                            "Assignment": {
                                "type": "object",
                                "properties": {
                                    "Id": {
                                        "type": "string",
                                        "description": "Assignment-Id"
                                    },
                                    "Target": {
                                        "type": "object",
                                        "properties": {
                                            "ConnectedOrganization": {
                                                "type": "object",
                                                "properties": {
                                                    "Id": {
                                                        "type": "string",
                                                        "description": "Assignment-Target-ConnectedOrganization-Id"
                                                    },
                                                    "DisplayName": {
                                                        "type": "string",
                                                        "description": "Assignment-Target-ConnectedOrganization-DisplayName"
                                                    },
                                                    "Description": {
                                                        "type": "string",
                                                        "description": "Assignment-Target-ConnectedOrganization-Description"
                                                    }
                                                }
                                            },
                                            "Id": {
                                                "type": "string",
                                                "description": "Assignment-Target-Id"
                                            },
                                            "ObjectId": {
                                                "type": "string",
                                                "description": "Assignment-Target-ObjectId"
                                            },
                                            "DisplayName": {
                                                "type": "string",
                                                "description": "Assignment-Target-DisplayName"
                                            }
                                        }
                                    },
                                    "State": {
                                        "type": "string",
                                        "description": "Assignment-State"
                                    },
                                    "Status": {
                                        "type": "string",
                                        "description": "Assignment-Status"
                                    },
                                    "AssignmentPolicy": {
                                        "type": "object",
                                        "properties": {
                                            "Id": {
                                                "type": "string",
                                                "description": "AssignmentPolicy-Id"
                                            },
                                            "DisplayName": {
                                                "type": "string",
                                                "description": "AssignmentPolicy-DisplayName"
                                            }
                                        }
                                    }
                                }
                            },
                            "Requestor": {
                                "type": "object",
                                "properties": {
                                    "Id": {
                                        "type": "string",
                                        "description": "Requestor-Id"
                                    },
                                    "ObjectId": {
                                        "type": "string",
                                        "description": "Requestor-ObjectId"
                                    },
                                    "DisplayName": {
                                        "type": "string",
                                        "description": "Requestor-DisplayName"
                                    }
                                }
                            }
                        }
                    }
                },
                "operationOptions": "IncludeAuthorizationHeadersInOutputs"
            }
        },
        "actions": {
            "Get_Resource_assigned_to_Package": {
                "runAfter": {},
                "type": "Http",
                "inputs": {
                    "uri": "https://graph.microsoft.com/beta/identityGovernance/entitlementManagement/accessPackages/@{triggerBody()?['AccessPackage']?['Id']}?$expand=accessPackageResourceRoleScopes($expand=accessPackageResourceRole,accessPackageResourceScope)",
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
            "Parse_Access_package_response": {
                "runAfter": {
                    "Get_Resource_assigned_to_Package": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_Resource_assigned_to_Package')",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "accessPackageResourceRoleScopes": {
                                "type": "array",
                                "items": {
                                    "type": "object",
                                    "properties": {
                                        "accessPackageResourceRole": {
                                            "type": "object",
                                            "properties": {
                                                "originId": {
                                                    "type": "string"
                                                }
                                            },
                                            "required": [
                                                "originId"
                                            ]
                                        }
                                    },
                                    "required": [
                                        "accessPackageResourceRole"
                                    ]
                                }
                            }
                        },
                        "required": [
                            "accessPackageResourceRoleScopes"
                        ]
                    }
                }
            },
            "For_each_1": {
                "foreach": "@outputs('Parse_Access_package_response')?['body']?['accessPackageResourceRoleScopes']",
                "actions": {
                    "Get_a_row": {
                        "metadata": {
                            "013PAXKNPUBABUDUCKB5BLWCPSPTEP6TPS": "/AccessPackageLookup/ApplicationIDtoDGPERM.xlsx"
                        },
                        "type": "ApiConnection",
                        "inputs": {
                            "host": {
                                "connection": {
                                    "name": "@parameters('$connections')['excelonlinebusiness']['connectionId']"
                                }
                            },
                            "method": "get",
                            "path": "/drives/@{encodeURIComponent('b!hab2joo7DU2VbEF8XnhY316PX-DE_0RFv5PSU44RgzPmV0KBvw04SJS9RYZeuhVB')}/files/@{encodeURIComponent(encodeURIComponent('013PAXKNPUBABUDUCKB5BLWCPSPTEP6TPS'))}/tables/@{encodeURIComponent('{6CFB91FA-5743-4C6E-BA01-5747A04F2142}')}/items/@{encodeURIComponent(encodeURIComponent(items('For_each_1')?['accessPackageResourceRole']?['originId']))}",
                            "queries": {
                                "source": "sites/mngenvmcap899704.sharepoint.com,8ef6a685-3b8a-4d0d-956c-417c5e7858df,e05f8f5e-ffc4-4544-bf93-d2538e118333",
                                "idColumn": "ApplicationID"
                            }
                        }
                    },
                    "Condition": {
                        "actions": {
                            "Is_Removed_Stage": {
                                "actions": {
                                    "Set_Permissions": {
                                        "type": "Http",
                                        "inputs": {
                                            "uri": "https://graph.microsoft.com/v1.0/oauth2PermissionGrants",
                                            "method": "POST",
                                            "body": {
                                                "clientId": "@{body('Get_a_row')?['ApplicationID']}",
                                                "consentType": "Principal",
                                                "resourceId": "@{body('Get_a_row')?['ResourceID']}",
                                                "principalId": "@{triggerBody()?['Assignment']?['Target']?['ObjectId']}",
                                                "scope": "@{body('Get_a_row')?['Permissions']}"
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
                                    }
                                },
                                "else": {
                                    "actions": {
                                        "Get_permissions_PER_user_Per_APP": {
                                            "type": "Http",
                                            "inputs": {
                                                "uri": "https://graph.microsoft.com/v1.0/oauth2PermissionGrants?%24filter=clientid+eq+'@{body('Get_a_row')?['ApplicationID']}'+and+principalId+eq+'@{triggerBody()?['Assignment']?['Target']?['ObjectId']}'",
                                                "method": "GET",
                                                "authentication": {
                                                    "type": "ManagedServiceIdentity",
                                                    "audience": "https://graph.microsoft.com/"
                                                }
                                            },
                                            "runtimeConfiguration": {
                                                "contentTransfer": {
                                                    "transferMode": "Chunked"
                                                }
                                            }
                                        },
                                        "Parse_individual_ODIC_PERM": {
                                            "runAfter": {
                                                "Get_permissions_PER_user_Per_APP": [
                                                    "Succeeded"
                                                ]
                                            },
                                            "type": "ParseJson",
                                            "inputs": {
                                                "content": "@body('Get_permissions_PER_user_Per_APP')",
                                                "schema": {
                                                    "type": "object",
                                                    "properties": {
                                                        "value": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "object",
                                                                "properties": {
                                                                    "id": {
                                                                        "type": "string"
                                                                    }
                                                                },
                                                                "required": [
                                                                    "id"
                                                                ]
                                                            }
                                                        }
                                                    },
                                                    "required": [
                                                        "value"
                                                    ]
                                                }
                                            }
                                        },
                                        "For_each": {
                                            "foreach": "@outputs('Parse_individual_ODIC_PERM')?['body']?['value']",
                                            "actions": {
                                                "Delete_permissions_4_odic": {
                                                    "type": "Http",
                                                    "inputs": {
                                                        "uri": "https://graph.microsoft.com/v1.0/oauth2PermissionGrants/@{items('For_each')?['id']}",
                                                        "method": "DELETE",
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
                                                }
                                            },
                                            "runAfter": {
                                                "Parse_individual_ODIC_PERM": [
                                                    "Succeeded"
                                                ]
                                            },
                                            "type": "Foreach"
                                        }
                                    }
                                },
                                "expression": {
                                    "and": [
                                        {
                                            "not": {
                                                "equals": [
                                                    "@triggerBody()?['Stage']",
                                                    "AssignmentRemoved"
                                                ]
                                            }
                                        }
                                    ]
                                },
                                "type": "If"
                            }
                        },
                        "runAfter": {
                            "Get_a_row": [
                                "Succeeded",
                                "Failed"
                            ]
                        },
                        "else": {
                            "actions": {
                                "Compose_1": {
                                    "type": "Compose",
                                    "inputs": "@body('Get_a_row')"
                                }
                            }
                        },
                        "expression": {
                            "and": [
                                {
                                    "not": {
                                        "equals": [
                                            "@body('Get_a_row')?['Permissions']",
                                            "@null"
                                        ]
                                    }
                                }
                            ]
                        },
                        "type": "If"
                    }
                },
                "runAfter": {
                    "Parse_Access_package_response": [
                        "Succeeded"
                    ]
                },
                "type": "Foreach"
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
            "value": {
                "excelonlinebusiness": {
                    "id": "/subscriptions/b91f57c6-f572-4f58-a857-e0953c9c895d/providers/Microsoft.Web/locations/eastus/managedApis/excelonlinebusiness",
                    "connectionId": "/subscriptions/b91f57c6-f572-4f58-a857-e0953c9c895d/resourceGroups/Logic_App_Automation/providers/Microsoft.Web/connections/excelonlinebusiness",
                    "connectionName": "excelonlinebusiness"
                }
            }
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
ble Data Reader access" -ForegroundColor Green

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

