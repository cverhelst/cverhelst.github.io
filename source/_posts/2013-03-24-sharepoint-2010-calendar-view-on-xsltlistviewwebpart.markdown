---
layout: post
title: "SharePoint 2010 - Calendar view on XSLTListViewWebPart"
date: 2013-03-24 09:49
comments: true
categories: 
- sharepoint
- fixes
---

Recently I tried changing the View of an __XSLTListViewWebpart__ to a _calendar_ view from code but it didnt work.

I tried it the same way as I did it for every other webpart, 
setting the __ViewGuid__ property, but it kept using a normal list view of its events, which wasn't even its default (which was the calendar). 
When I investigated the issue further I noticed something strange. 

When setting the webpart's View to the Calendar view from the GUI, it worked, but it also changed the webpart from an __XsltListViewWebpart__ to an ordinary __ListViewWebpart__. 

	SPList list = ...
	SPView view = ...
	
	ListViewWebPart wp = new ListViewWebPart();
	...
	wp.ViewGuid = view.ID.ToString("B").ToUpper();

When I created a ListViewWebpart instance in code and set its View to the calendar view it now worked!
