---
layout: post
title: "Powershell - Delete a WebPart from a page"
date: 2012-10-30 22:46
comments: true
categories: 
- powershell
- sharepoint
---

When deleting a WebPart from a page with _PowerShell_ you need to get the _WebPartManager_ from that page and call the __Delete()__ method.

There is one thing you have to keep in mind in case of pages in libraries with _content approval_. Here, it is important you _check-out_ the page __before__ getting the _WebPartManager_ for the page. Otherwise PowerShell will complain that the page is checked out by someone else (even though it's really you who checked out the page).