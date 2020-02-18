---
layout: post
title: Another reason to hate SharePoint
---


A common interview question for SharePoint professionals is to ask the candidate what they hate about the product. This is a great interview question because it's fun, it's open-ended, and it allows people to open up about their experiences with SharePoint. And let's face it, there's a lot to hate about SharePoint.

I have a long list of things I hate about SharePoint, and today I encountered a new one - in Modern sites, when using a page template, calculated default field values to not get applied to the page.

In the Classic SharePoint publishing world, we could use page layouts to define the look and feel for new pages as they are created. Page layouts sucked in a lot of ways - they were scoped to site collections so they were difficult to update tenant-wide, they had really convoluted metadata associations, and they were tightly bound to content types. But they did allow us to specify a pre-defined look and feel to new pages.

Modern SharePoint did away with this, disallowing content types and any notion of page layouts (at first). This was a big regression in functionality, and customers complained, so Microsoft relented, adding Content Type support back in, and providing a mechanism to templatize pages for re-use.

But it was half-assed, probably because it was an afterthought. Microsoft put a Templates folder in Site Pages and provided a solution whereby a page author could copy a page from that folder into the root directory and provision a page from there.

This has downstream after-effects: it copies all metadata from the source page, ignoring default values. Which wouldn't be so bad, except for calculated default values, because those will propagate the values calculated from the template page instance.

In my example I had an Expiration field which was supposed to default to one year from today (**=TODAY+365** in calculated field lingo). I can provision this field from schema, and the default setting works great, except when I use a page template. With a page template it uses the creation date of the template file, so the dates are always off.

Is this really a big deal? No, I suppose not. It's inconsistent, but anyone who works with SharePoint knows that the only thing consistent is inconsistency.  We can work around this and come up with a manageable solution to the requirement. But I shouldn't have to. Default settings should "just work" and page templates shouldn't break other supported functionality.

Microsoft should have anticipated that templatized pages were going to be a required feature, and should have built that into the product. It's just sloppy, inarticulate architecture, and creates more work for me, and less value for my clients.
