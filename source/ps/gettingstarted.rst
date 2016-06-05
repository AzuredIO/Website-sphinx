Installing Azure Powershell.
============================

The official way of installing Azure PowerShell is via the `Web Platform Installer <https://www.microsoft.com/web/downloads/platform.aspx>`_ Running the installer will automatically install required prerequisites. This should install on any Windows OS from XP forward.


A quicker way to install Azure PowerShell is directly from the `Github repository <https://github.com/Azure/azure-powershell/releases>`_ This also ensures that you have the latest version installed since the Github releases are the first release point.

Installing Microsoft Online Services Module (MSOL)
--------------------------------------------------

In order to manage Azure Active Directory, the MSOL module needs to be installed. This connects directly to Azure Active Directly and manages users and groups etc.

There are two parts to installing MSOL, first the `sign in assistant <https://www.microsoft.com/en-gb/download/details.aspx?id=28177>`_ and then the `module itself <https://msdn.microsoft.com/en-us/library/jj151815.aspx>`_. Once these are installed we can log in.

Logging in
----------

The easiest way to log into Azure PowerShell is to simply use::

    $account = Add-AzureRmAccount

This will provide a browser pop up window that you can enter your credentials in to and log in. Whenever you log into Azure PowerShell your session will only last for the duration of the PowerShell session. When you close the window you will log out of Azure.

To enable easier logging in you can store your Azure session token to a local file that you can then reuse to log in with.::



      Save-AzureRmProfile -Profile $account -Path e:\ps\azure.json


To then log back in to Azure you can use::


      Select-AzureRmProfile -Path E:\ps\azure.json


Azure Account Types
-------------------


There are two types of accounts that can be used to log into an Azure account. Either a Windows Live ID or an Azure Active Directory account. When you log into a Windows Live account there are a number of redirects needed to obtain a token from the live.com servers. These redirects have to be done from the client and can only be done through a web browser.

When you log into a Live ID account through PowerShell it will initiate a web browser to take care of that process. Unfortunately this process removes the ability to log in to Azure non-interactively. It is also not possible to log in to MSOL with anything but an Azure Active Directory account.

In order to fully use Azure from within Powershell you need to create an Azure Active Directory account. The problem we have is that it is not possible to create an account from Powershell.

##Creating an Azure Active Directory Account

Currently it is only possible to create an AAD account from the [old portal](manage.windowsazure.com) (we're told that Active Directory management is coming to the new portal soon)

Adding a custom domain name
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. Note:: This part is entirely optional, it simply makes for less typing. 

When you create a new user account you will need to log in with the username@{tenant}.onmicrosoft.com. To simplify this you can add a custom domain name to Azure, so you can log in with username@example.com

To add a custom domain, navigate to the Active Directory section, select the 'Default Domain' (presuming you haven't changed it) and go to the 'Domains' tab. Click on 'Add'

You will then be prompted for the domain name (which you must already own) and on the next page you will be given a code that you add to your domain as a txt record. Once this is done you can verify ownership of the domain and it will be associated with your subscription.

Adding a user
^^^^^^^^^^^^^

In the Users tab of the Default Domain there is an 'Add user' at the bottom of the page. Click this and select 'New user in your organization' for 'type of user', give them a user name, and if you added one make sure that the right domain name is being used. On the next page you set their name and display name. Since this is going to be our primary account to manage other users and accounts with. Set the role as Global Admin, when you select this you will also need to specify an email address for the account.

You will then need to open a new browser session, I usually use private browsing for this step, and log in to the account with the username you just created, and the password you were given. You will then be able to assign a new password to the account.

At that point we are ready to log into Azure PowerShell with our new account.

###Logging in again

From a new Azure PowerShell session, you can now log in to both Azure and MSOL with the following code.::

    $cred = Get-Credential
    Add-AzureRmAccount -Credential $cred
    Connect-MsolService -Credential $cred


This piece of PowerShell stores our login credentials into the $cred variable and then passes that to both Azure Resource Manager and MSOL. However if we try to log into Azure Service Management using the `Add-AzureAccount` we will get an error. This is because the Global Administrator is a Resource Management permission, and not a Service Management one. We will explore the distinction between those later.

Saving Credentials
^^^^^^^^^^^^^^^^^^

While being able to use the -credential parameter for logging in is useful, it still has the problem that password tokens will expire and you will need to reauthenticate occasionally.

If you are on a secure machine, it is possible to use your user profile to securely store a password, you can wrap a small script around this to allow you automatically log into Azure.

The Windows Data Protection API (DPAPI) uses a hash of the user's password to create the encrypted data. This means that if you use this method of securing a password then that Azure account will only be as strong as the local machine password. Microsoft recommend using a strong password if you are storing secrets in DPAPI.

First we create a secure string object with the password we want to encrypt.::

    $securePassword01 = ConvertTo-SecureString "pass@word1" -AsPlainText -Force


Then we can convert that to an encrypted string and save it to disk.::

    $securePassword01 | ConvertFrom-SecureString | Out-File e:\ps\securepass.txt


We then need to read that from disk, and create a credential object.::

    $securePassword02 = Get-Content E:\ps\securepass.txt | ConvertTo-SecureString
    $credentials = New-Object System.Management.Automation.PSCredential ("admin@example.com", $securePassword02)


You can then verify the password::

    $credentials.GetNetworkCredential().password

You can put that into a small script that you can call whenever you need to log into Azure, and save yourself having to log in regularly. Again you need to evaulate the security risk of doing that.
