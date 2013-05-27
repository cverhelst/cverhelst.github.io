---
layout: post
title: "SharePoint 2010 - Export Built-In ListViewWebPart"
date: 2013-05-27 16:47
comments: true
categories: 
- sharepoint
---

In SharePoint 2010 you can export WebParts by editing the page they are on and in their Edit Menu, selecting "Export...". However, this does not work for the built-in ListViewWebparts for the lists.

But, there is way of enabling it. Every WebPart has a property called __ExportMode__ that determines if a WebPart is exportable or not. It's value is an Enum Type of [System.Web.UI.WebControls.WebParts.WebPartExportMode] and can have a couple of values like 'None' and 'All'.

Setting it on the ListViewWebPart with PowerShell is rather easy:

	$SiteUrl = "http://mysite"
	$ListName = "My List"
	
	$Web = Get-SPWeb $SiteUrl

	$Page = $Web.GetFile("/Lists/" + $ListName + "/LastModified.aspx")
	$WPM = $Page.GetLimitedWebPartManager([System.Web.UI.WebControls.WebParts.PersonalizationScope]::Shared)
	
	$WP = $WPM.WebParts[0]
	$WP.ExportMode = [System.Web.UI.WebControls.WebParts.WebPartExportMode]::All

	$WPM.SaveChanges($WP)

	$Page.Update()
	
Enjoy!