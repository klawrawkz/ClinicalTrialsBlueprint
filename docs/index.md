---
page_type: sample
languages:
- csharp
products:
- dotnet
description: "Add 150 character max description"
urlFragment: "update-this-to-unique-url-stub"
---

# Clinical Trials Matching Service Blueprint


### Requirements
* Clone/Download this repository to you local drive
* [Install the Azure PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-3.3.0)


### Connect to Azure Subscription
```PowerShell
Login-AzAccount

Set-AzContext -Subscription <Your Subscription Name>
```

### FHIR Server
Create a Resource group for the FHIR server. It must be in a separate resource group.

```PowerShell
$fhirRg = New-AzResourceGroup -Name ctm-fhir-blueprint -Location eastus
```
Assign Fhir Server Name
```PowerShell
$fhirServiceName = <fhir service name>
```

Create the Fhir server deployment. You will to provide a admin password for the SQL server

```PowerShell
New-AzResourceGroupDeployment -ResourceGroupName $fhirRg.ResourceGroupName -TemplateFile ..\arm-templates\azuredeploy-fhir.json -serviceName $fhirServiceName
```

Verify that the Fhir Server is running

```PowerShell
$metadataUrl = "https://$fhirServiceName.azurewebsites.net/metadata" 
$metadata = Invoke-WebRequest -Uri $metadataUrl
$metadata.RawContent
```
It will take a minute or so for the server to respond the first time.

### Text Analytics for Healthcare
Create Resource Group for the that will contain all the resources required for the blueprint

```PowerShell
$rg = New-AzResourceGroup -Name ctm-blueprint -Location eastus
```

Assign the name for the Text Analytics for Healthcare service
```Powershell
$ta4hServiceName = <text analytics service>
```
Create the TextAnalytics for Healthcare deployment
```PowerShell
New-AzResourceGroupDeployment -TemplateFile ..\arm-templates\azuredeploy-ta4h.json -ResourceGroupName $rg.ResourceGroupName -serviceName $ta4hServiceName
```

Verify that Text Analytics for Healthcare service is running
```Powershell
$statusUrl = https://$ta4hServiceName-ayalon-webapp.azurewebsites.net/status
$status = Invoke-WebRequest -Uri $statusUrl
$status.RawContent
```
It will take about 20 minutes for the service to deploy and run

### Matching Service

Assign strcuturing service name
```Powershell
$matchingServiceName = <ctm matching service>
```

Create Matching Service deployment
```Powershell
New-AzResourceGroupDeployment -TemplateFile ..\arm-templates\azuredeploy-matching.json -ResourceGroupName $rg.ResourceGroupName -serviceName $matchingServiceName -textAnalyticsService $ta4hServiceName
```

Check that the query engine is up and running
```Powershell
Invoke-WebRequest -Uri https://$matchingServiceName-qe-webapp.azurewebsites.net/matching
```

### Healthcare Bot
Assign the Healthcare Bot service name 
```Powershell
$botServiceName = "ctm-bot"
```
Create the Healthcare Bot SaaS Application
```powershell
$saasSubscriptionId = .\marketplace.ps1 -name $botServiceName -planId free
```

Deploy Healthcare Bot resources

```powershell
.\default_azuredeploy.ps1 -ResourceGroup $rg.ResourceGroupName -saasSubscriptionId $saasSubscriptionId  -serviceName $botServiceName -botLocation US
```
This command can take few minutes to complete.


