---
layout: post
title: "SharePoint 2010 Search - Setting the default Scope"
date: 2012-11-13 20:13
comments: true
categories: 
- sharepoint
- search
- fixes
---

Setting the search scope automatically to a specific scope on a web-by-web basis is not so obvious, even a default scope. I found there are several ways to do it though.

The first, and the easiest, is to configure the Core Results WebPart on the results pages of the Search Centers. These will have a property where the default scope can be set.

The second is probably the most flexible. When you search using a search box anywhere on a site, it will send your "keywords" (what you typed in the searchbox) to this results page in the url. There is a parameter, 's', that allows setting the scope for the search. If your search scope has a name with spaces in it you'll have to escape it with '%20'

    ..../searchresultspage.aspx?k=myfirstkeyword&s=MyScope

An added benefit is that this one will overrule the default scope of the Core Results WebPart (from the first option I explained).

A third way I found was the AppQueryParameters of the SmallInputSearchBoxDelegate. You'll need to do this in the codebehind in one of the events (OnInit worked for me). Careful where you adjust it [1](#issue).

    this.AppQueryParameters = "Scope=\"My Scope\""

The AppQueryParameters allow to filter on a bunch of different things but one of them is the search scope. If your search scope name has spaces, you'll need to put double quotes around it, but otherwise it is not mandatory.

<span id="issue" />
1: If you run into strange issues with saving / checking in / not being able to change webpart settings on a page containing this delegate, make sure you did not set the AppQueryParameters in a wrong method. For example, this behavior will happen when you set it in the CreateChildControls() method.