---
layout: post
title: "SharePoint 2013 - Internal Javascript Classes"
date: 2014-12-17 20:22
comments: true
categories: 
- sharepoint
- javascript
- scriptsharp
---

Recently, we had a very strange issue with some customization failing and blocking built-in functionality.

The customization aside, we noticed this strange difference between our SharePoint environments.

![SP JS files with different content on identical servers]({{ root_url }}/assets/images/sharepoint/2013/sp_tileview_js_different_properties.png)

So basically, these JS files are the same, yet the obfuscated member names of the class are different across environments (not all).
The last modified dates of the files of one server were actually different from the other, but apart from this they were the same. Our other 2 environments had the same member names as 1 of these.

The reason for the obfuscated Javascript is because Microsoft generates these files using [Scriptsharp](https://github.com/nikhilk/scriptsharp).

The fact that they're generated may be the reason they can be different. I don't think they're generating during the installation of your SP Farm, but that would explain why they can have different member names across environments.
What I do think is more likely, is that they generate it on their side and it gets deployed as is to everywhere. But possibly the generation algorithm was changed and ended up naming these members differently between versions.

I still don't get how both environments are supposedly the same SP Farm version, but oh well.

So let this be a warning if you intend to interact with these generated classes (or overwrite their functionality... as our offending customization did -_-) as you can't be certain that the naming is gonna remain the same over time or be the same across environments.
