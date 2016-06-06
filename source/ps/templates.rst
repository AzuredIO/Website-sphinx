Resource Manager Templates
==========================

A core component behind Azure Resource Management is the template. This is a json document that describes the deployment you are wanting to create. Each resource template is scoped to a resource group. So you can only create a deployment for a single resource group.

A basic template to deploy a storage account might look like this
::

 {
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "resources": [
     {
       "type": "Microsoft.Storage/storageAccounts",
       "name": "storageaccountname01",
       "apiVersion": "2015-06-15",
       "location": "westeurope",
       "properties": {
         "accountType": "Standard_LRS"
       }
     }
   ]
 }

You could then call this from PowerShell and deploy your storage account.::

  New-AzureRmResourceGroupDeployment -Name test1 -ResourceGroupName testgroup `
                                     -TemplateFile .\test.json

Obviously you won't always want to deploy that exact storage account, so we can include parameters by changing the following

To save space, we've stripped out the extraneous lines.::


    "parameters": {
     "StorageAccountName": {
       "type": "string",
       "defaultValue": "mystorageaccount01",
       "metadata": {
         "description": "Unique DNS Name for the Storage Account"


        "type": "Microsoft.Storage/storageAccounts",
        "name":  "[toLower(parameters('StorageAccountName'))]",

Now we deploy our script with the following PowerShell.
::
  
    New-AzureRmResourceGroupDeployment -Name test1 -ResourceGroupName testgroup `
                                       -TemplateFile .\test.json `
                                       -StorageAccountName StorageAccountName01

By using the 'toLower' function when we specify the name we can ensure that the name will match Azure Storage Account naming conventions.
It is also possible to add a default parameter, so that parameter can be missed out of the commandline altogether.

You can provide a prepoulated list of allowed vaules, to limit the options that are available. For instance we could set the accountType like this
::

  "storageAccountType": {
  "type": "string",
  "defaultValue": "Standard_LRS",
  "allowedValues": [
    "Standard_LRS",
    "Standard_GRS",
    "Standard_ZRS"
  ],

This will allow you to use tab completion in PowerShell, and prevent any other storage account type from being used.

For
