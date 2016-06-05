Resource Groups
===============

If you deploy a single virtual machine on Azure, you will create six separate components. These would be:

#. Storage Account
#. Virtual Network
#. Network Interface
#. Public IP Address
#. Security Group
#. Virtual Machine.

Now imagine you're deploying a solution that has ten web servers. You are suddenly in the situation where you will require around forty components (since the virtual network and storage account can be shared). This can become a huge burden to manage.

Azure's solution to this is the Resource Group. This creates a logical container to group all of those components together in.

Azure Resource Management Templates.
------------------------------------

Part of the philosophy behind the move to ARM was the concept of creating all related infrastructure in a single deployment. From the previous example of creating a virtual machine. In PowerShell it would take approximately ten discrete commands to create that virtual machine. If one of those components fail it can be a time consuming task to manually clean up and run the scripts again.

The ARM approach to this is to create a json document that describes the entire deployment. That document will contain all of the configuration information required to deploy a complete solution to Azure. So if you want to create a twenty node Docker cluster you can have a single document that will describe the entire deployment.

You can also deploy that same template multiple times under the same Azure Subscription, simply by using a different Resource Group name. This means you can create a test environment that exactly matches a deployed environment.

Resource Group Boundaries
-------------------------

There is no reason that you couldn't create a single resource group that would contain your entire infrastructure. Every service you consume from Azure could be created from the same template and with a single command you could deploy every server and service that exists. At the other end of the spectrum there is no reason that you couldn't put every single component into its own resource group. You could have a single resource group for each of the six items listed at the top of this page.

I'm sure you see why both of those scenarios would be unmanageable for any size of infrastructure. However, it does raise the question of where should you create boundaries between resource groups. what factors should you consider when deciding what should go into which resource group.

Considering that a resource group is primarily a deployment tool it should be used to bind resources that have a shared lifecycle. That is, resources that are likely to be created and destroyed together. So if you have a virtual network that contains Active Directory and database servers and is connected to your on premises network. Then you should look at how you can group those components. It should be considered when each group might need to be redeployed. For instance if the database servers are upgraded it is unlikely that you would also need to redeploy the Active Directory Servers. You also couldn't deploy either the Active Directory or the database servers in the same group as the virtual network, because both would depend on that resource. So for this scenario you would need three resource groups.

Other scenarios are less obvious though, if you have a group of web servers, some middleware servers and a database server. The argument can be made that they all have the same lifecycle and should be created in the same resource group. Equally there are good arguments to be made for putting each in its own group. There are other considerations you can use to decide the boundaries of your resource groups.

Complexity
^^^^^^^^^^

The first consideration is the complexity of the template. A resource template is a difficult format to read and understand what is happening. It is possible to have a template that spans multiple files with each being called under different sets of parameters. While these very large templates are impressive they also introduce scope for problems later on. They can be difficult to debug when they don't work as expected. A better solution can be to split those templates into multiple resource groups and have a script that manages deployment and dependencies.

Security
^^^^^^^^

One of most useful aspects of Resource Groups is the ability to use them as a security boundary. It is possible to assign user and group permissions at the Resource Group level. So in the above example it would be possible to have DBA's granted permissions over the database servers, and restricting their access to the network or Active Directory Servers. Equally network admins could be granted permissions only to the network security group.

Conclusion
----------

Resource Groups and Resource Group Templates are an excellent way to manage multiple resources, they allow you to deploy, delete and redeploy solutions in a consistent, reliable manner. They provide a simple mechanism to secure infrastructure and provide principles of least privilege to deployments. They greatly simplify the management of infrastructure. In that light it makes sense to keep the templates themselves as simple as possible. If only for the sake of your sanity when you have to debug them in six months' time.
