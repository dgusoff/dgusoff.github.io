---
layout: post
title: Connect to an AAD Secured web API from SharePoint Framework
---


A while back I had a requirement to connect to an Azure AD-secured API (Azure Function) from a SharePoint Framework web part, and I found it to be difficult and a little frustrating to get it working. There just seemed to be a lot of points that were non-obvious and a little "magical", at least to me, but I was able to figure it out. I started a blog post on this topic, because I knew thie requirement would come around again, and I didn't want to incur that learning tax again, but I never got around to finishing the post.

Well, the requirement has come up again and I'm finding myself treading the same unfamiliar waters as last time, so I thought I'd wrap up the blog post here so I don't run into this issue again.

It's true that there are already a bunch of blog posts out there on this very topic, but I wasn't able to find one that could help me solve all the problems. Some had conflicting information, and some had information that just seemed wrong. Maybe the tech has changed since those were published, and maybe it'll change again after I publish this. As always in tech, your mileage may vary.

## High level steps

In this post I'll cover the steps needed to create an Azure AD-secured API and connect to it from a SharePoint Framework web part. At a high level the steps look like this:
1. Create the Azure Active Directory App registration
2. Create the API using an Azure Function
3. Secure the API using the AAD app, and test
4. Code the web part and test on the workbench

At this point we'll enter the world of API permissions, and encounter some of the common stumbling blocks you might encounter while doing this, and dicuss solutions.

5. Configure API permissions
6. Configure CORS
7. Test locally
8. Deploy and profit


## Create the Azure AD application

In this post I'm going to assume that your Azure AD is not necessarily in the same Azure tenant as your Azure API. While it's probably  the norm for production scenarios to have both the AAD and Azure resources in the same Azure tenant, it probably isn't for developer scenarios. In my case I have an MS Demo tenant for my SharePoint environment and AAD and my MSDN Azure subscription to host my API.

To create the AAD application we're going to run an Azure CLI command to create an app called "demo-spfx-api":

#-> consider adding a gist here instead of inline code

````az ad app create --display-name demo-spfx-api````

(If you're using a SharePoint tenant that does not have an Azure subscription in the same directory, you'll need to include the allow-no-subscriptions flag when you log in: ````az login --allow-no-subscriptions````)


Note, there are some configurations we'll want to add to this but I'm leaving them out at this time so we can see what kind of errors we get when we omit them, and then we'll address the configurations.




















------ old stuff
Having the ability to connect to secure APIs from Modern SharePoint Online provides opportunities to create greater value by connecting to secure data in a cloud-friendly manner.  As you might expect, there is some complexity to work out, and there are a number of blog posts out there explaining this, but none of them was able to get me all the way through to a successful solution. Maybe the infrastructure has changed subtly, maybe the posts assumed knowledge I didn't have, maybe they were happy-path MVPs glossing over the tricky parts, I don't know. But I felt the world could use one more blog post to explain how to do this.

In this post, I'll explain how I was able to deploy an Azure-hosted, Azure AD-secured API, and then consume that API in a SharePoint Framework web part. One quirk of my solution is that the API and the Azure AD application do not reside in the same Azure tenant. I suspect a lot of developers have a situation similar to mine, where I have an Azure subscription, and a SharePoint Online developer tenant, but they do not live inside the same Azure environment. This blog post takes care to address that situation.

# Disclaimer

It's May 6, 2019 as I write this. You'll be reading this sometime in the future. Things might change in the meantime. Things might break. That's life in the cloud.

# Assumptions

In order to follow along at home I'm going to assume you are able to do the following things:
1. You can deploy an API to Azure, either in Functions or a web app service. Platform and language do not matter here.
2. You can create, code, and deploy a SharePoint Framework web part.
3. You have at least a passing familiarity with Azure Active Directory appliations

# High level steps

We're going to follow the following steps to make this work:
1. Define the API in Azure
2. Create the Azure AD applicaiton
3. Secure the API using the AAD application and verify functionality
4. Write the web part
5. Set up the API permissions in SharePOint
6. Test and verify


# Create the Azure API

This part is really just a formality as I mentioned in the assumptions you should be able to do this. In my example, we'll just create an Azure Function running C# that returns a simple JSON array from a GET request.

--code sample

At this point, you should be able to run the GET in the browser with no authentication and get a result back:

- screen shot


# Create the Azure AD application

Now, while logged in as the administrator of your SharePoint Online tenant, navigate to portal.azure.com. If you're using a dev tenant, you probably won't be able to create any resources here, but you should be able to find the Azure Active directory backing your tenant, and be able to create applications on it.

-screen shots

other instructions


# Secure the API

# Write the Web Part

# Configure API permissions

# test and verify