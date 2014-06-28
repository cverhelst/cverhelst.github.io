---
layout: post
title: "PowerShell - Search Crawl History"
date: 2014-06-13 17:16
comments: true
categories: 
- sharepoint
- search
- powershell
---

I've been working with Search in SP2013 lately and maintaining search involves looking in the Search Administration pages, that admittedly give you a lot of information. Sometimes I'd like an easier way of getting it, though.

Sadly, some of the things that the Search Administration UI shows you, like the Crawl History, do not have a direct PowerShell equivalent.

After reflecting on the logic behind the page, I came up with this:
```Powershell
	$numberOfResults = 10
	$contentSourceName = "MyContentSource"
	
	[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.Office.Server.Search.Administration")
	
	$searchServiceApplication = Get-SPEnterpriseSearchServiceApplication
	$contentSources = Get-SPEnterpriseSearchCrawlContentSource -SearchApplication $searchServiceApplication
	$contentSource = $contentSources | ? { $_.Name -eq $contentSourceName }
	
	$crawlLog = new-object Microsoft.Office.Server.Search.Administration.CrawlLog($searchServiceApplication)
	$crawlHistory = $crawlLog.GetCrawlHistory($numberOfResults, $contentSource.Id)
	$crawlHistory.Columns.Add("CrawlTypeName", [String]::Empty.GetType()) | Out-Null
	
	# Label the crawl type
	$labeledCrawlHistory = $crawlHistory | % {
	 $_.CrawlTypeName = [Microsoft.Office.Server.Search.Administration.CrawlType]::Parse([Microsoft.Office.Server.Search.Administration.CrawlType], $_.CrawlType).ToString()
	 return $_
	}
	
	$labeledCrawlHistory | Out-GridView
```
Ow yes folks, **Microsoft.Office.Server.Search.Administration.CrawlLog.GetCrawlHistory** is a ***public*** method (I was almost sure I'd run into an internal one when reflecting the dll). You give it the number of results you want to get and the contentsource id (which is a simple int number actually).

The first two variables are parameters further down in the script. You show the crawl log for one contentsource specifically, and you **have** to specify how many results you want to return (0 returns no results, and -1 doesn't work, as it get's parameterized directly into the SQL statement, giving you a nice error that SELECT TOP N or some such can't be negative).

I'm assuming you're running in a *SharePoint Management Shell*. I load the assembly through the deprecated *Reflection* method, because frankly, it's the only one that works consistently (I'm looking at you **Add-Type**).

There's some logic in there to show the crawl type by it's **string** representation, instead of the **int** that is inside the DataTable of results.

At the end, I pipe the results to **Out-GridView** which you need to have the PS ISE installed for I think. And you should, as it's a nice way to display table results. And the editor gives good intellisense too :).

Enjoy!
