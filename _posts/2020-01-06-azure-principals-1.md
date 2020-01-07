---
layout: post
title: Working with Service Princpals in Azure
---

Sometimes when we work with Azure we use automated processes and tools to interact with cloud resources there. These processes and tools need a way to authenticate to Azure to do their work. Often they will use a service principal to do this.

A service principal is an identity, similar to a user identity you'd use to log into the portal, but different in many ways. A service principal is optimized for use by automated, or unattended applications, so you wouldn't use one the way you'd use your user account - by logging into the web portal, for example. Rather, they'd be used by back-end processes to do stuff on your behalf.

You've probably already created some service principals in your Azure environments, even if you don't realize it. If you've ever run a DevOps pipeline into Azure, if you've ever deployed an application from Visual Studio or VS Code, or if you've ever accessed the Azure Resource Manager actions from a Flow or Logic App, then you've created service principals under the covers to handle authentication from these tools into Azure.

We can view existing service principals in the Azure Portal by going to subscription -> Azure Active Directory -> App Registrations -> All Applications. 

![screenshot](/images/az-principals/1.png "Screen shot")


In the screen shot above, the principals with trailing guids were created by Visual Studio and Azure Devops. The others are just Azure AD app registrations I created for other things. This brings up an important point: Azure service principals are just Azure AD app registrations with some role assignments applied.

## Role Assignments

A service principal doesn't mean much unless it has role assignments granted to it. The actual permission to take action is defined in role assignments.

You can view existing role assignments in the Azure portal at subscription -> access control -> Role Assignments. 

![screenshot](/images/az-principals/2.png "Screen shot")

Take note of the Role and Scope columns in the table above. A role assignment consists of three parts: a principal (user or app), a Role (a choice from a predefined list of roles), and a scope (the thing being accessed).

If you click "Roles" on the Access Control page, you'll see the list of predefined roles. There are a couple hundred predefined roles that are scoped to specific resource types and operations - for example, CDN Endpoint Contributor, and CDN Endpoint Reader - to allow you to specify exactly the permission you want to grant. It's best practice to be as specific as you can, only granting the principal what it absolutely needs to accomplish its task. There are also more general roles, such as "Reader", "Contributor", and "Owner", that you can assign to your scope.

The scope of a role assignment can be a specific resource, a resource group, or an entire subscription. Again, you want to be restrictive by default, so only assign the scope the principal really needs to access. If your principal needs to create resource groups, it'll need to be a contributor on the entire subscription, but if it is simply reading data from Application Insights, it can get by with the Monitoring Reader role on the specific resource.

## Many ways to create Azure Service Principals

There are a number of ways to create Azure Service principals. The Azure CLI and the Az PowerShell module have commands to do exactly this. I do not prefer creating service principals this way for production use because these methods create principals with the Contribute role against the entire subscription by default. It's a much better idea to use the third method: create the Azure AD app registration and explicitly add the role assignments you need. It does take a little longer to do it this way but it encourages us to use best practices and think hard about what we are doing while we're doing it. It also forces us to think ahead about what roles and scopes we really need to add to our app.

If you're still interested in creating Azure service principals using the command line tools you can check out the documentation:

[Azure Service Principal via CLI](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)

[Azure Service Principal via PowerShell](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-powershell)

## Create an Azure Service Principal in the portal

I'll be mostly following [this documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal) to create the service principal. In your Azure subscription, go to Azure Active directory and create a new App Registration. You can leave out the redirect URI since this won't be a web app. Once the app is created, take note of the Application ID, because we'll use that in our integrations.

![screenshot](/images/az-principals/3.png "Screen shot")

 Once that's created, go to the "Certificates and Secrets" blade and create a new client secret. Give it a description and set an expiration policy and click Add. The secret will generate and display on the page. Make sure to copy this value since it won't appear again. We'll need it to complete our integrations.

![screenshot](/images/az-principals/4.png "Screen shot")

## Add Role Assignments

Now we have a service principal in Azure but it can't do anything yet, because it has no permissions. We'll use Role-Based Access Control (RBAC) in Azure to assign permissions to Azure Resources. In my case I want the principal to be able to read all resource groups, so I'll need to assign the "Reader" role at the Subscription scope. We can assign role assignments at the resource, resource group, or entire subscription level.

Navigate to the asset in question and click Access Control (IAM). Click Add a Role Assignment, find your principal by name in the search box, and click "Save". Your principal now has access to Azure, at the exact role and resource level needed to do its job.

![screenshot](/images/az-principals/5.png "Screen shot")

We can use the Role Assignments tab to review existing role assignments on that resource, and modify and remove as needed.

In this post we reviewed Azure Service Principals and Role Assignments, created them using the Azure Portal. In part 2 I'll explore some ways we can use this principal to programmatically interact with Azure.
