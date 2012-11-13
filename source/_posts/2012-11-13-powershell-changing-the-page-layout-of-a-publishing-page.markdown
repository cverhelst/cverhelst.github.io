---
layout: post
title: "PowerShell - Changing the Page Layout of a publishing page"
date: 2012-11-13 19:08
comments: true
categories: 
- sharepoint
- powershell
- pagelayout
---

When changing the page layout of a publishing page in SharePoint, you need to be careful that the page is of the same contenttype that is associated with the pagelayout. Otherwise you will get errors when loading the page.

Firstly, there are several ways to get hold of the pagelayout you mean to change to.

- Through the _SPPublishingSite Object_:
<!-- -->
    $PageLayoutName = "MyPageLayout"
    $PSite = New-Object Microsoft.SharePoint.Publishing.PublishingSite($Web.Site)
    # GetPageLayouts(<SPContentTypeObject>,<RemoveObsolete>)
    $Layouts = $PSite.GetPageLayouts($CType, $true)
	$PageLayout = $Layouts | ? { $_.Name -eq $PageLayoutName }
    
- If this does not get you the Page Layout you were looking for, there is a more round about way of getting it by going straight to the Master Page Gallery holding all the Page Layouts ( and Master Pages, obviously ):
<!-- -->
    [Microsoft.SharePoint.SPList]$MPList = $Site.GetCatalog([Microsoft.SharePoint.SPListTemplateType]::MasterPageCatalog)
    $PageLayout = $MPList.RootFolder | ? {$_.Name -eq $PageLayoutName}
	$PageLayout = New-Object Microsoft.SharePoint.Publishing.PageLayout($PageLayout.Item)
    
Now that you've got the Page Layout Object, all that's left to do is to assign it to the Publishing Page.

    $PubPage.Layout = $PageLayout
    $PubPage.Update()
    
And that's that.