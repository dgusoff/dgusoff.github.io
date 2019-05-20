---
layout: post
title: Connect to an AAD Secured web API from SharePoint Framework
---

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