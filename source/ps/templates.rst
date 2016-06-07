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
         "storageAccountType": "Standard_LRS"
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


By using the 'toLower' function when we specify the name we can ensure that the name will match Azure Storage Account naming conventions.
It is also possible to add a default parameter, so that parameter can be missed out of the commandline altogether.

Now we can deploy that template with the following PowerShell.
::

    New-AzureRmResourceGroupDeployment -Name test1 -ResourceGroupName testgroup `
                                       -TemplateFile .\test.json `
                                       -StorageAccountName StorageAccountName01


For parameters like storageAccountType you can provide a prepoulated list of allowed values, to limit the options that are available.
By setting the defaultValue we can omit it all together, and it will default to that value.
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

For properties like apiVersion we could use a parameter with a default value, that would allow us to override that value in future.
But since most templates are targeted at a specific API it makes sense to keep the API version as a constant. However copying the value into every section of a template will be error prone.
::

  "variables": {
  "apiVersion": "2015-06-15"
  },
  --------------------------
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "[variables('apiVersion')]",

The most useful function of variables is to construct runtime variables, based on either parameters or other runtime variables. For instance, you could have constructions like these.
::

    "location": "[resourceGroup().location]",
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'storage')]",

The first of these will set the location to whatever the resource group's location is set to.
The second creates a hash of the resource group id, and appends 'storage' on the end to create a unique storage account name.
There are many functions within templates for modifying strings and parameters to make them useful elsewhere in templates.
There will be more of them covered as we progress through this series.
