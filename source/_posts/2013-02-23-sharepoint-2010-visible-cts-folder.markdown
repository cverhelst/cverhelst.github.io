---
layout: post
title: "SharePoint 2010 - Visible \_cts folder"
date: 2013-02-23 00:19
comments: true
categories: 
- sharepoint
- fixes
---

We had an issue recently that occured when deploying a list definition with a feature. It had a <contenttyperef> element that contained a <folder> element with a target (?)attribute. The target was set to "\_cts/My ContentType Name". The issue was that it was creating \_cts folders in all of the document libraries deployed with a similar list definition. 

After some research online we noticed some things.

- The cts folder was not supposed to be visible(duh). Having it visible in the document library with file explorer meant it had been copied.
- The \_cts folder should seemingly only exist at the site collection level, in the root. We were seeing it in document libraries in subsites as well.
- Several online examples showed a different approach to the target attribute. Some used the Content type name 'as is'. Others used the \_cts folder as a prefix, but with a leading slash. Others still suggested using ~site as a prefix.

Especially the last point meant there might be an issue with a relative path being interpreted as an absolute.
We didn't get around to testing the ~site prefix but merely deleting the \_cts prefix alltogether fixed the issue for us. We did briefly try with a leading slash but that seem to work.
