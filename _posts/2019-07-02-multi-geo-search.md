---
layout: post
title: Search-driven customizations on Multi-Geo SharePoint Online. 
---

I recently had a client enable Multi-Geo functionality in SharePoint Online in order to satisfy some regulatory requirements, and they were finding that the search-driven experiences we had built for them did not seem to be multi-geo aware, and asked me to come up with a solution.  Having never seen multi-geo in the wild before I jumped at the chance to tackle an *interesting problem* and set about looking for a solution.

## What's Multi-Geo?

Multi-Geo is a licensing feature of SharePoint that allows organizations to split its SharePoint Online tenant into distict logical units with data stored in different geographic regions. This is often done in order to comply with regulations like [GPDR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation).

Multi-Geo is set up so that there will be a main tenant, called a "Central Location", and auxiliary tenants called "Satellite Locations". In many ways the satellite locations look and behave like discrete tenants. They will have different admin portals and different domain names in the site URLs. They have separate settings for search schema and app catalog. They also seem to subscribe to separate content type hubs, although they share a common taxonomy.

Multi-Geo is enabled at a user level, and each user can be configured to have their data stored in the desired location. You can set it up like this:

`Get-MsolUser -UserPrincipalName dgusoff-eu@contoso.onmicrosoft.com | Set-MsolUser PreferredDataLocation 'EU'`

Another offshoot of the user's configured location is that teams and sites created by the user will be housed automatically in the appropriate location, even if initiated from a different location.

## Implications on Search
Does Search respect multi-geo locations? Well, it depends. Delve and the SharePoint page will be multi-geo aware right out of the box. The Search Center and any REST API-based custom apps will not, although they allow some configuration options. [This documentation](https://docs.microsoft.com/en-us/office365/enterprise/configure-search-for-multi-geo) has more detail on Multi-Geo implications on Search.

## Search Results Web Parts

Search Results Web Parts will not find data from other geo locations by default, but they can be configured to do so by checking the "Show Multi-Geo results" on the web part settings.

![alt text](/images/multi-geo/search-results.png "Web Part settings")

## REST API Search queries

Calls to the Search REST API will not pull results from other locations by default, but they can be configured to do so by adding the appropriate parameters to the query string of a GET request, or to the body of a POST request. The REST approach affers more flexibility in that it supports querying a subset of the locations if you'd like.

### non-geo-aware query

```http
/_api/search/query?querytext='sharepoint'
```

### REST query with multi-geo support

```http
/_api/search/query?querytext='sharepoint'&Properties='EnableMultiGeoSearch:true'&ClientType='my_client_id'
```


The ClientType parameter is a means of identifying different applications and its value can be literally any valid parameter string. You cannot omit it however and still get multi-geo results.


### Fetching a subset of locations

You can specify a subset of geo locations in your queries by adding a JSON object into your query URL. I haven't done this since "my" tenant only contains two locations, but here's the syntax. You can read more on this [here](https://docs.microsoft.com/en-us/office365/enterprise/configure-search-for-multi-geo).

```http
/_api/search/query?querytext='site'&ClientType='my_client_id'&Properties='EnableMultiGeoSearch:true, MultiGeoSearchConfiguration:[{DataLocation\:"NAM"\,Endpoint\:"https\://contosoNAM.sharepoint.com"\,SourceId\:"B81EAB55-3140-4312-B0F4-9459D1B4FFEE"}\,{DataLocation\:"CAN"\,Endpoint\:"https\://contosoCAN.sharepoint-df.com"}]'
```

## Other configurations to be aware of

As stated above, different geo locations have different app catalogs, so you'll need to deploy your solutions to multiple places. This will complicate your support efforts somewhat. Same goes for your search configurations and content type hub, if you are unfortunate enough to be one of the few folks still using that.


## Read more

[Overview of Multi-Geo](https://docs.microsoft.com/en-us/office365/enterprise/office-365-multi-geo)

[Search documentation](https://docs.microsoft.com/en-us/office365/enterprise/configure-search-for-multi-geo)
