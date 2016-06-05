A brief history of Azure
========================

ASM vs ARM
----------

Azure launched in 2009 as a SaaS solution to provide web and data services. Azure PowerShell was released in late 2009. Azure was launched with strong reliance on the concept of a cloud service that hosted underlying services, providing port management and NAT services to the resources hosted within.

When Virtual Machines launched in 2012 it was natural for those machines to occupy the same cloud services as the SaaS offerings that came before them.

In 2012 the concept of infrastructure as code was just beginning to become important to unfortunately Azure didn't work well in this space as it was difficult to deploy solutions at scale without very complicated scripts, and close to impossible to have a completely hands off approach to server configuration.

Security within Azure at this stage was an all-or-nothing approach. Making a user a co-administrator to the entire subscription was the only way to grant privileges, meaning that many companies needed to create multiple subscriptions as the only way to manage security.

This was the Azure Service Management framework (ASM), and it very clearly had some substantial issues.

Azure's answer to this came at Build 2014 when they announced the new portal and Azure Resource Manager (ARM). This allowed the grouping together of resources under a single management unit so they could be deployed and destroyed as a single unit. That management unit also allowed a security boundary to be created so permissions could be bound solely to that one management unit. It was also possible to create templates to define the entire group of resources, so they could be consistently and predictably deployed. This was the start of resource groups and Azure Resource Management.

However this did mean that there are now two distinct management frameworks that have no visibility of each other. This presented a problem to Azure PowerShell, because there needed to be a way to distinguish between ASM and ARM.

PowerShell ARM
--------------

.. sidebar:: Site Focus

  Because of the rapidly deteriorating support for ASM, this site will focus on ARM whenever possible. 

Azure PowerShell's answer to this was to provide a context switch between the two frameworks.
The downside of this was that the same cmdlets would give very different results depending whether they were run in the ASM or ARM space.
This was finally resolved by splitting out the Azure Resource Manager cmdlets and renaming them with the AzureRm scheme.


ARM Namespace
-------------

.. todo:: this needs expanding / moving once a more suitable place is written

Under the covers the PowerShell ARM cmdlets communicate with Azure using the Rest Interface. The rest namespace has a simple format that it is useful to learn and understand.

For example a storage account would have the following format.
::

    /subscriptions/{subscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Storage/storageAccounts/{AccountName}

and a Virtual Machine would have
::

    /subscriptions/{subscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VMName}

This namespace can then used to set permissions on each level. So you could put a wildcard after the providers and set a permission on everything in a resource group. Or by setting a wild card on the resource group name you could set permissions for all virtual machines in all resource groups.
