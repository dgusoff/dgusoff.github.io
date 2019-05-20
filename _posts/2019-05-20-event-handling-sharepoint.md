---
layout: post
title: Responding to events in SharePoint Online - Techniques and Considerations
---

In SharePoint, things happen, and often we need to trigger other things in response to those events.

As is so often the case in SharePoint, there are several different ways to handle this scenario, and the decision of which approach to use depends on the specifics of the situation. In this blog post I'll discuss what those options are and my opinions on their strengths and weaknesses.

## Remote Event Receivers
Because of the legacy of event receivers in on-premises SharePoint, Remote Event Receivers are sort of the default when most people think of event handling in SharePoint Online. And in my opinion RERs still rule the roost. They offer the most flexibility in terms of the events they handle, they are able to be deployed at mass scale, and since they run code, they can do pretty much anything.

RERs do have the puzzling attribute of having been architected as WCF endpoints. I mean, even in 2013 this was incredibly archaic. They also do not benefit from being tied to the bizarre App/Add-in model, which greatly adds to the difficulty in developing, debugging, and deploying. I wrote [a blog post](https://derekgusoff.wordpress.com/2017/11/08/remote-event-receivers-youre-all-doing-it-wrong/) back in 2017 about how to side-step this frustration and develop an "app-less" remote event receiver, check it out if you're inclined.

Remote Event Receivers offer [dozens and dozens](https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-server/jj167297(v=office.15)) of event triggers, making them by far the most flexible of all the apraches discussed in this post. They also are the only way to create a *synchronous* event handler, meaning they halt the user interface and return their result back to the user. You can use this, for example, to conditionally cancel events like deletions.

Power: **High** . You can do anything that can be expressed in code.

Effort: **High**. You need to write code, and you also need to host your code in an internet-callable WCF service.

Scalability: **High**. You can programmatically attach remote event receivers. You can add them to PnP provisioning templates. You can use the same endpoint to serve thousands of different instances.

Trigger Options: **High**. As mentioned above you have many trigger options at your disposal, both synchronous and asynchronous.

------------------------------------

## SharePoint WebHooks
Webhooks were introduced with much hype and promised to simplify the experience developers were having with WCF-based Remote Event Receivers, being more amenable to modern develpoment patterns such as REST.  Enthusiasm for SharePoint WebHooks has waned, at least in my estimation, when their deficiencies became evident.

In theory, setting up a webhook seems pretty straightforward. Create a subscription on a resource, configuring an endpoint, and then coding that endpoint to handle the incoming notifications.

Webhooks launched with support for asynchronous list item events, and to this day that is all they support. Also, maddeningly, webhook subscriptions expire, and the maximum allowable duration is 180 days. So any implementation of webhooks would have the additional overhead of checking for exipiration, and re-registering the webhook if necessary. The one project I worked on that used webhooks actually made this re-registration every single time it was invoked, I mean because why not? 

Webhooks also do not record much detail on the event that triggered it, unlike remote event receivers. So you'd also have to query the list item to figure out exactly what changed.

Power: **High** . You're writing custom code here, so you can do literally anything. You do have the additional overhead of managing subscription expiration and querying for changes.

Effort: **High**. You need to write code, and you also need to host your code in a web API somewhere. Also, that somewhere needs to be reachable from SharePoint Online. The cloud is a good choice for htis.

Scalability: **High**. You can programmatically create webhook subscriptions, so you can bulk add them by the thousands or include them in an automated provisioning process, no problem.

Trigger Options: **Medium**. You're limited to list items and async trigger. The full list of triggers is [here](https://docs.microsoft.com/en-us/sharepoint/dev/apis/webhooks/lists/overview-sharepoint-list-webhooks#list-event-types)

-----------------------------

## Flow / Logic Apps
I'm adding Logic Apps to this because they do mostly the same things as Flow, but for all practical purposes I'm really just talking about Flow.  Flow is the new Microsoft no-code productivity platform, and it is a real game-changer. Flow offers a huge range of functionality in an easy-to-use browser-based format. I'm finding that I use Flow for a lot of things I would have written custom code for in years past.

Power: **High** . In addition to the ever-growing list of triggers and actions provided within Flow, you also have the ability to easily interact with other services. You can call HTTP endpoints (including other Flows exposed as such) or you can drop an item on an Azure Queue, for example, for further prcessing by a WebJob or Function.

Effort: **Low**. Flow really is easy to use, although lots of complex logic branching or looping will complicate the design surface. You can do a lot, though, before it starts to get cumbersome.

Scalability: **Low**. Flows are one-off solutions and cannot be easily duplicated or automated. They also have no concept of version history, so you can't easily revert to a working version should development efforts go amock. Logic Apps, however, are JSON-based so they can be versioned and scaled, although each deployed Logic App would be a separate resource in Azure.

Trigger Options: **Medium**. Flow's SharePoint triggers are mostly item-based, but there's a lot of them. Also, Flows can be set up with HTTP triggers so they can be used in creative ways.

---

## Timer Jobs
Not really a pure event-handling scenario, but you can poll for changes and take action when a change is discovered. This is more of a "pull" approach rather then the "push" the other options feature. You might find this useful for events that are not explicitly suported by the official means. For example:
1. When a site collection is created
2. When something changes in the term store
3. Anything else you want to support

Essentially a timer job is a custom app that you write, that runs unattended on a schedule. This means you have lots of control over how it's coded and hosted. It can be a PowerShell or Bash script that runs on a local computer, or it could be an Azure WebJob or Function. It could even be a Flow or Logic app that initiates the job.

Power: **High** . This is custom code, so anything goes. Remember, that power comes with great responsibility.

Effort: **Medium/High**. Depending on your requirements, it could be pretty simple, or hugely complex.

Scalability: **Low**. Your code and/or hosting will have to handle that scalability for you.

Trigger Options: **Undefined**. There are no "triggers" here, you're doing all the work yourself.

---

## SharePoint Designer Workflows
I really don't recommend anyone do this but I'll include it here for completeness. In addition, this might be the only viable option for some unfortunate folks out there who do not have access to Flow or the ability to deploy code.

SharePoint Designer has been around since at least the 2007 version of SharePoint, and for years it's been used to create interesting business solutions without having to write or understand code. It's always been notorious for its flaky behavior and difficulty to maintain. SharePoint Designer 2013 was the last version that will be released, so its days are numbered, although it can still be used.

Power: **Low** . You are limited to the set of aptions provided by the tool. You don't have any ability to add custom logic, although the ability to call an HTTP Web Service allows you to craft SharePoint REST calls or interact with custom services.


Effort: **Low**. With a GUI-based interface, SPD workflows are pretty straightforward to author, even though the tool is buggy and crash-prone.

Scalability: **Low**. SharePoint Designer has no concept of replicating or deploying workflows to different locations. 

Trigger Options: **Low**. You can author workflows to un against items in lists or libraries. You can set them to trigger when items are created or updated, and you also have the option to launch them manually.


---

## My Recommendation
If I had a new requirement to respond to an event today, my recommendation would depend on a couple of questions: Do I need to handle an event in one place or in many places? What is the event I'm trying to capture?

If I needed to be able to handle events in may places - for example, thousands of team sites - then I would write a remote event receiver. If the logic was sufficiently complex, I'd use the RER to drop an item on an Azure Queue and let a WebJob do the actual processing. WebJobs give us a degree of fault tolerance and scale that web endpoints just don't have, so that's why I'd add that extra layer if it made sense to do so.

If I had a one-off situation and Flow could handle the trigger I'd probably write a Flow to consume the event. If the logic was easily doable in Flow I would just implement the whole thing in Flow. If it were more complex, again, I'd use the Flow to drop an item onto an Azure Queue and let the WebJob do its thing.

Have any questions or comments? Write them up below! 

