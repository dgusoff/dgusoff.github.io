In [part 1](https://dgusoff.github.io/azure-principals-1/) of this blog I discussed Service Principals and Role Assignments in Azure and walked through how to set them up. In this blog we'll put the concepts to work by creating a couple of simple integrations against Azure.

## Part 1 recap

In part 1 we created a service principal and assigned it some permissions. In order to build out an integration we need to note the following items from part 1:

- The tenant ID of the Azure subscription (a guid)
- The Subscription ID (a guid)
- The Application ID of the Azure AD application (yep, another guid)
- The client secret of the application (a long string, not a guid)

The three guids we can reclaim easily enough from the portal. The client secret is only exposed when it's created, so if you missed your chance to save it, go ahead and delete it and recreate another one.

## Using HTTP requests and the raw REST API

Once we have a service principal set up, it's a pretty straightforward two-step process. First, we use an [OAuth Client Credentials Grant](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) to obtain an access token. Then we use that token as a bearer token against the REST endpoint we want to call.

## The client credentials grant

I'll be using Visual Studio code and the [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) extension to send and receive my REST calls. This is quite a bit simpler than either Fiddler or Postman, and is super easy to use.

Our first request will be a POST to the 

````
POST https://login.microsoftonline.com/<tenant-guid>/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=<application-id>&client_secret=<app-secret>&resource=https://management.azure.com/

````

When we execute this in REST Client, and you've hooked everything up properly, you'll get a response like this:

![image.png](/images/az-principals/6.png)


You'll see the response includes the token along with some information about the token, including its expration time. Tokens are good for an hour after they're generated. If you get anything other than a 200 response, you've done something wrong.

If we've obtained an access token, we can use that to authenticate against the Azure REST API. In my example I'll query for a list of Resource Groups. My call will look like this:

````
GET https://management.azure.com/subscriptions/<subscription-guid>/resourcegroups?api-version=2019-08-01
Authorization: Bearer <access_token>
````

Don't forget the space between "Bearer" and the token. Also, remember to add the role assignment to a=enable your principal to access the resources. If you've wired it up properly you'll get a 200 response and a JSON collection of your resource groups.

![image.png](/images/az-principals/7.png)

Now that you understand how to authenticate against the Azure REST API with the service principal, you should be able to implement a solution against Azure using any language.  But there are also APIs you can use to simplify your Azure integrations.

## Using the Azure Fluent API with a Service Principal

The Azure Fluent ..NET API is really simple to use, and you can use it along with your service principal to connect to Azure.

To get started, create a .NET Core Console Application and add the `Microsoft.Azure.Management.Fluent` Nuget package. Add the following code to your Main method:

```` C#
 IAzure azure = Azure.Authenticate(@"C:\Creds.json").WithDefaultSubscription();            

 var rgs =  azure.ResourceGroups.List();
 foreach(var rg in rgs)
 {
    Console.WriteLine(rg.Name);               
 }
````
The JSON file contains a representation of the principal and subscription data, in the format:

````
{
  "clientId": "b52dd125-9272-4b21-9862-0be667bdf6dc",
  "clientSecret": "ebc6e170-72b2-4b6f-9de2-99410964d2d0",
  "subscriptionId": "ffa52f27-be12-4cad-b1ea-c2c241b6cceb",
  "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
````

There are other authentications spelled out in the [documentation](https://docs.microsoft.com/en-us/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet), including Key Vault or configuration objects.

If you Fiddle the traffic while running this app, you'll see in the Fiddler trace the exact two HTTP requests for requesting the token and executing the API call.

There are Azure SDKs for Python, Node, and Java along with .NET. Once we understand the basics of how to authenticate with Service Principals we can use any of these languages to build functionality against Azure.