---
layout: post
title: "SharePoint Calculated Column - Calendar Start Time"
date: 2012-11-17 00:10
comments: true
categories: 
- sharepoint
- fixes
---

If you want to use the [Start Time] field of Calendar lists in a calculated column at a level above this list, you'll run into some strange issues. I came across this weirdness when I wanted to created a Site Column containing a formula with this field.

The problem is that Calendar lists have a field _[Start Time]_ that does not exist at the Site Column level, making it rather hard to create __calculated columns__ with formulas based on this field at the Site Column level. At least, through the GUI or PowerShell, because those will trigger a syntax validation that will check if the column used exists or not, and this will fail.

Strangely, there is not the case for the [End Time] field, as there is an equivalent at the Site Column level.

There are a few solutions, depending on the manner in which you want to deploy the Site Column. 

## _First solution:_

__You deploy from Fields elements.xml__.

Here, you can simply keep using the name of the column as it appears in the List, [Start Time]. For some reason, syntax validation is not performed when the Site Column is created with a formula that supposedly contains an unexistant column.

However, this does not work when creating the column through the GUI or PowerShell at the Site Column Level, because here the syntax validation does get triggered. But onto solution two:

## _Second solution:_

__You need to deploy it from the GUI or PowerShell__.

In this case, you can use a different field name. It's very strange, but somehow the [Start __Date__] field (which _does_ exist at the Site Column level) is automatically translated by SharePoint to [Start __Time__], from the _Site Column level_ to the _List level_. This means that you can create the calculated field with a formula using [Start Date] as a Site Column. And when it is used at the List level, SharePoint will have translated the formula field reference to [Start Time].

This only works if you deploy through the GUI or PowerShell, though. The translation does not automatically occur when the field was deployed through the Fields elements.xml. Should this not work for some reason, I've also come across the field name [EventDate] (note the missing space). I've had some luck using this field name as well, as it is sometimes being translated to this one internally.

One way to find out for sure is to check the schema xml of the calculated field at both the List level and the Site Column level through a tool like SharePoint Manager or PowerShell.
