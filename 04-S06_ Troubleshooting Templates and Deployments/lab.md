---
# This basic template provides core metadata fields for Markdown articles on docs.microsoft.com.

# Mandatory fields.
title: Lab - Troubleshooting Templates and Deployments
description: 115-145 characters including spaces. This abstract displays in the search result.
author: CHDAFNI-MSFT
ms.author: CHDAFNI # Microsoft employees only
ms.date: 8/21/2019
ms.topic: AzureTemplate-Lab
# Use ms.service for services or ms.prod for on-prem products. Remove the # before the relevant field.
# ms.service: service-name-from-white-list
# ms.prod: product-name-from-white-list

# Optional fields. Don't forget to remove # if you need a field.
# ms.custom: can-be-multiple-comma-separated
# ms.reviewer: MSFT-alias-of-reviewer
# manager: MSFT-alias-of-manager-or-PM-counterpart
---
# Lab - ARM Templates - Troubleshooting Templates and Deployments

>[!ALERT]Please note that the lab environment is restricted by policy. You will be able to create only the Azure resources required by the lab.



This lab will guide you through troubleshooting a variety of template deployment failures that you may encounter in the real world.

The templates have already been written and need to be carefully reviewed to determine what is causing their deployment failure. The templates can be found in `C:\Lab_Files\M04\S06` along with an error-free version of the templates. Also, there will be links throughout the lab that can be clicked to reveal the answer. 

Remember that each Azure resource is typically deployed in its own operation and therefore can be a very useful method for troubleshooting. To see each operation's request & response (for helping determine which area of the template is erroring out) use the following example:
> [!NOTE] This will only work if the deployment is submitted to ARM. For pre-validation errors, the deployment will not exist.

1. Sign into @lab.VirtualMachine(StandardVM).SelectLink as +++@lab.VirtualMachine(StandardVM).Username+++ using +++@lab.VirtualMachine(StandardVM).Password+++ as the password.
1. The deployment must use the flag `-DeploymentDebugLogLevel All`
   1. Example:

      ```powershell
      New-AzResourceGroupDeployment -Name PIP -ResourceGroupName '@lab.CloudResourceGroup(1851).Name' -TemplateFile ".\pip.json" -PIPCount 10 -Verbose -DeploymentDebugLogLevel All
      ```

1. Pull back each operation by referencing the original deployment's name
   1. Example:

      ```powershell
      $operations = Get-AzResourceGroupDeploymentOperation -DeploymentName PIP -ResourceGroupName '@lab.CloudResourceGroup(1851).Name'
      
      foreach ($operation in $operations)
      {
          Write-Host $operation.id
          Write-Host "Request:"
          $operation.Properties.Request | ConvertTo-Json -Depth 10
          Write-Host "Response:"
          $operation.Properties.Response | ConvertTo-Json -Depth 10
      }
      ```



## Lab 1
1. Deploy `Lab1.json` and when prompted, enter values of your chosing.
1. Did the deployment fail and if so, why did it fail? If the deployment succeeded, can you find ways the deployment would have failed?

    > [!hint] Check for template validation failure message. It will detail if the values provided were invalid and why.

1. ^[Click for the answer][Lab1Answer1]

    >[Lab1Answer1]:
    >Check the parameter defnitions in the template for allowed value restrictions.
    >1. The `StorageAccountName` parameter has `maxLength` set to 7. If the value passed is longer than 7 characters, template validation will fail.
    >1. Storage accounts names must use numbers and lower-case letters only. If the value passed to the `StorageAccountName` parameter contains an upper-case letter or a special character, template validation will fail.
    >1. The `DepartmentTag` parameter has a set of `allowedValues`. If the value passed is not one of the allowed values, template validation will fail.

1. Attempt to redeploy `Lab1.json` using valid values

## Lab 2
1. Deploy `Lab2.json`
1. Why did the deployment fail?

    > [!hint] The error should return the JSON path with the line and character position that failed parsing.

1. Review the template file in VS Code and use the Problems panel at the bottom (or use Ctrl+Shift+M shortcut) to view issues VS Code has identified
1. Remediate all of the JSON structure issues in the template and redeploy. ^[Click for the answer][Lab2Answer1]

    >[Lab2Answer1]:
    >1. The `allowedValues` for the `DepartmentTag` parameter multiple issues
    >   1. There is a trailing comma missing from `"HumanResources"`
    >   1. `IT` needs to be wrapped in double-quotes
    >
    >   ```json
    >   "allowedValues": [
    >       "Engineering",
    >       "HumanResources",
    >       "IT"
    >   ]
    >   ```
    >
    >1. The name property value for the virtualNetwork resource on line 39 is missing a closing bracket
    >
    >   ```json
    >   "name": "[concat(parameters('StorageAccountName'),'vnet')]",
    >   ```
    >
    >1. The name property for the networkInterfaces resource on line 69 has two issues:
    >   1. A colon must be placed between the property name and value
    >   1. The value must be wrapped in double-quotes
    >
    >   ```json
    >   "name": "networkInterface1",
    >   ```
    >
    >1. The dependsOn property for the networkInterfaces resource on line 77 is missing a closing bracket
    >
    >   ```json
    >   "[resourceId('Microsoft.Network/virtualNetworks', concat(parameters('StorageAccountName'),'vnet'))]"
    >   ```
    >
    >1. There is an extra closing brace on line 93 that should be removed

1. With the errors resolved, attempt to redeploy `Lab2.json`

## Lab 3
1. Review `Lab3.json`. Will you be prompted for parameters?
1. Deploy `Lab3.json`
1. Why did the deployment fail?

    > [!hint] This time, the underlying error is not surfaced in the initial error. You will have to dig into the inner errors further down to find the cause of the first failure.

1. How can the error be resolved for all future deployments? ^[Click for the answer][Lab3Answer1]

    >[Lab3Answer1]:
    >If using the default paramter values, the deployment fails because because the storage account name starts with 'Storage' and storage accountnames     must be lower-case only. You can ensure the storage account name is always lower case using the `toLower()` function.
    >
    >1. Define a new variable for the storage account name and use the `toLower()` function to convert the `StorageAccountName` parameter to lower     caseregardless of what was entered. Example:
    >
    >   ```json
    >   "variables": {
    >       "StorageAccountName": "[toLower(parameters('StorageAccountName'))]"
    >   }
    >   ```
    >
    >1. With the new variable defined, references to the original `StorageAccountName` parameter must be updated to use the variable instead of the     parameter.
    >   1. There are two references, under the name property for the storageAccounts and virtualNetworks resources. Example:
    >
    >   ```json
    >   "name": "[variables('StorageAccountName')]",
    >   ```
    >
    >   1. There is a third reference, under the subnet id of the networkInterfaces resource on line 86. Example:
    >
    >   ```json
    >   "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', concat(variables('StorageAccountName'),'vnet'), 'Subnet-1')]"
    >   ```

1. Resolve the error and attempt to redeploy `Lab3.json`
1. Why did the deployment fail?

    > [!hint] Because pre-deployment validation succeeded, you can use the Get-AzResourceGroupDeploymentOperation script at the beginning of the guide to gather deployment errors. Were all dependent resources deployed in time? How can resource dependency be defined?

1. ^[Click for the answer][Lab3Answer2]

    >[Lab3Answer2]:
    >The deployment failed because the network interface attempted to provision before the subnet was created. If you attempt to redeploy the template again, it will succeed because the subnet is now created. To correct the issue so it deploys properly the first time, the network interface needs to have a dependsOn property applied:
    >
    >```json
    >"name": "networkInterface1",
    >"type": "Microsoft.Network/networkInterfaces",
    >"apiVersion": "2018-08-01",
    >"location": "[resourceGroup().location]",
    >"tags": {
    >    "displayName": "networkInterface1"
    >},
    >"dependsOn": [
    >    "[resourceId('Microsoft.Network/virtualNetworks', concat(variables('StorageAccountName'),'vnet'))]"
    >],
    >"properties": {...}
    >```

1. Resolve the error and attempt to redeploy `Lab3.json`

## Lab 4
1. Deploy `Lab4.json`
1. Why did the deployment fail? ^[Click for the answer][Lab4Answer1]

    >[Lab4Answer1]:
    >The SQL database is missing a dependency on the SQL server
    >
    >```json
    >{
    >	"type": "databases",
    >	"apiVersion": "2018-06-01-preview",
    >	"name": "[variables('databaseName')]",
    >	"location": "[parameters('location')]",
    >	"sku": {...},
    >	"kind": "[concat('v12.0,user,vcore',if(contains(variables('sqlDatabaseServerlessTiers'),parameters('sqlDatabaseSkuName')),',serverless',''))]",
    >	"properties": {...},
    >	"dependsOn": [
    >		"[variables('sqlServerName')]"
    >	]
    >}
    >```

1. Resolve the error and attempt to redeploy `Lab4.json`
1. Why did the deployment fail? ^[Click for the answer][Lab4Answer2]

    >[Lab4Answer2]:
    >The web server farm resource has a dependency on the web site while the web site has a dependency on the web server farm. This creates a circular dependency. The web server farm should be created first and should not have a dependency on the web site. Remove the dependsOn property from the web server farm resource definition to resolve the circular dependency.
    >
    >```json
    >"type": "Microsoft.Web/serverfarms",
    >"apiVersion": "2018-02-01",
    >"name": "[variables('servicePlanName')]",
    >"location": "[parameters('location')]",
    >"properties": {...}
    >```

1. Resolve the error and attempt to redeploy `Lab4.json`

## Lab 5
1. Delete the App Service and Plan to prevent a conflict with Windows and Linux App Service Plans in the same Resource Group.
1. Deploy `Lab5.json`
1. Why did the deployment fail? ^[Click for the answer][Lab5Answer1]

    >[Lab5Answer1]:
    >Template validation failed because the web site config nested resource had an invalid segment length. Because the web site config was defined as a nested resource of the parent web site, it should not have the resource type fully qualified and should only reference config.
    >
    >```json
    >"apiVersion": "2016-08-01",
    >"name": "web",
    >"type": "config",
    >"dependsOn": [...],
    >```

1. Resolve the error and attempt to redeploy `Lab5.json`
1. Why did the deployment fail? ^[Click for the answer][Lab5Answer2]

    >[Lab5Answer2]:
    >Template validation failed because the subnet child resource name had an invalid segment length. Because the subnet was defined as a child resource outside of the parent virtual network, the name must reference the parent resource (VNet) in addition to the child resource (subnet)
    >
    >```json
    >"name": "Lab5vNet1/subnet1",
    >"type": "Microsoft.Network/virtualNetworks/subnets",
    >"apiVersion": "2018-08-01",
    >"dependsOn": [...]
    >```

1. Resolve the error and attempt to redeploy `Lab5.json`

## Lab 6
1. Deploy `Lab6.json`
1. Why did the deployment fail? ^[Click for the answer][Lab6Answer1]

    >[Lab6Answer1]:
    >The public IP address resource has a dependency on TailSpinPublicIPPrefix which is not a valid resource. Remove the dependsOn property.

1. Resolve the error and attempt to redeploy `Lab6.json`