---
layout: post
title: "CAML Queries"
date: 2012-10-24 20:44
comments: true
categories:
- caml
- sharepoint
---

CAML is a good way to query data in SharePoint if you care about performance since it will be run as an SQL query on SQL Server instead of just sending all the data over to you so you can do your own filtering.

The downside is that it's not very developer-friendly, as in, no Intellisense and arguably not so flexible or maintainable.

## Gotcha's

You can do some more advanced filtering in CAML if you want, but you have to know the language.

### Filter on field Id

Gotcha: The field GUID is specified with 2 capital letters: __ID__ and not Id

    <FieldRef ID='GUID'/>
    <Value Type='Text'>YourFilterValue</Value>  

### Filter on ContentTypeId

Gotcha: The field name is _ContentTypeId_, obviously, but so is the _Value Type_:

    <FieldRef Name='ContentTypeId'/>
    <Value Type='ContentTypeId'>0x00030230432432423040234023032</Value>
    
Although it is possible to specify Value Type as _Text_ or _Computed_ even, but then you can't do this when filtering:

    <BeginsWith>
        <FieldRef Name='ContentTypeId'/>
        <Value Type='ContentTypeId'>0x00030230432432423040234023032</Value>
    </BeginsWith>
    
That's right, your probably dealing with List Content Types when filtering so you need to somehow cope with this. If you specify your value with Value Type _ContentTypeId_ you can use the _BeginsWith_ comparator (strangely enough, _Contains_ does not work with Value Type _ContentTypeId_).