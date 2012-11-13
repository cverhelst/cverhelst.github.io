---
layout: post
title: "PowerShell - Changing the content type of an existing page or item"
date: 2012-11-13 19:07
comments: true
categories: 
- sharepoint
- powershell
- contenttype
---

__Before__ you change the content type of the item, be aware that you _may lose data_. Even if fields remain the same type it has been known to be reset when changing the content type, so please be careful to __backup any crucial data__.

Before changing the contenttype of your page, make sure that the content type you mean to change to is __associated with the list or library__. Otherwise you will get a generic error when loading your page.

You can do this simply by adding the _SPContentType Object_ to the list holding the item:
    
    $CType = $Site.RootWeb.ContentTypes[$ContentTypeName]
    $Item.ParentList.ContentTypes.Add($CType)
    
To actually change the content type of the item, you need to set the _ContentTypeId Property_ of the item and update the item.

    $Item["ContentTypeId"] = $CType.Id
    $Item.Update()
    
Note that calling the property like this:
    
    $Item.ContentTypeId = $CType.Id
    
will not work. PowerShell will complain that the property is _read-only_. Doing it the other way will work, though.