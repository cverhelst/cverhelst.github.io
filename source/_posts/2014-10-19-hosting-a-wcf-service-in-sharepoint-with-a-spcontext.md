---
layout: post
title: "SharePoint 2013 - Search - Content Enrichment - Basics"
date: 2014-10-19 15:07
comments: true
categories: 
- sharepoint
- wcf
- service host factory
- spcontext
---

## Hosting a WCF service in SharePoint with a SPContext

If you want to extend SharePoint, adding custom web services is one way to do it. There's several requirements usually, 
and if one of them is the necessity for a SPContext, you can use the built in [SharePoint WCF Service Factories](http://msdn.microsoft.com/en-us/library/office/ff521586.aspx).

There's a couple of things to keep in mind when using Service Host Factories. They're basically the programmatic alternative to the web.config configuration files. This has the benefit of not having to deploy or update a web.config file somewhere, but at the same time it has the downside of not even being able to quickly reconfigure something by putting a web.config next to the .svc file. It's all code from here, no configuration.

## Deployment

These .svc files can be deployed to the ISAPI hive folder, in their own subfolder preferably. They'll be accessible by this url from any site collection in the farm:

- http://{sitecollectionurl}/{subweburl}/_vti_bin/{pathToMySvcFileInTheISAPIFolder}/{myservicefilename}.svc
- _vti_bin points to the ISAPI folder, so {pathToMySvcFileInTheISAPIFolder} would be the hierarchy of folders you used inside the ISAPI folder before you see your {myservicefilename}.svc file.

It is "web" aware, so your SPContext.Current.Web will be whatever web you called the service at. Keep this in mind if you want to locate artifacts in your web / sites, because this would mean you'll have to call your service from the correct web to make them accessible.

Because the .svc file is deployed to the HIVE, you will need a SharePoint project in your solution to deploy the file with. This makes deployment itself really easy, you don't even have to activate a feature or create an IIS site. SharePoint takes care of everything.

## How do I create the service

A service factory is specified in the .svc file. SharePoint provides 3 service factories for you to use, depending on the kind of service you want to create:

- SOAP: MultipleBaseAddressBasicHttpBindingServiceHostFactory
- REST: MultipleBaseAddressWebServiceHostFactory
- ADO.NET Data Service : MultipleBaseAddressDataServiceHostFactory

I wanted to make a REST service, so I used the MultipleBaseAddressWebServiceHostFactory in my case. This means my .svc file looked like this: 

    <%@ServiceHost Language="C#" Debug="true"
    Service="NameSpaceOfMyService.MyServiceClassName, $SharePoint.Project.AssemblyFullName$"
    Factory="Microsoft.SharePoint.Client.Services.MultipleBaseAddressBasicHttpBindingServiceHostFactory, Microsoft.SharePoint.Client.ServerRuntime, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" %>
    
You'll notice we're using a VS Token in this file, to configure VS to replace the token in .svc files you can [follow the instructions described on MSDN](http://msdn.microsoft.com/en-us/library/office/ff521581.aspx#code-snippet-4).
    
For it to work correctly, you'll have to add (some extra attributes)[http://msdn.microsoft.com/en-us/library/office/ff521581.aspx#code-snippet-2] to the class implementation of your service:

- BasicHttpBindingServiceMetadataExchangeEndpointAttribute
- AspNetCompatibilityRequirements(RequirementsMode = AspNetCompatibilityRequirementsMode.Required)

## Custom service host factory

This means you're using the service as it's configured by SharePoint:

- 3 endpoints
  - /
  - /anon
  - /ntlm
- no /help page to browse to
- default 2mb upload limit


If you want to change these properties of your service, remember, placing a web.config near the .svc file will not make a difference. You'll have to subclass the service host factory and change the endpoints that have been created by SharePoint base class and update their settings in the __OnOpening__ event from the service host.

This also means you'll have to change the reference to the service host factory used by your service in the .svc file:

    <%@ServiceHost Language="C#" Debug="true"
    Service="NameSpaceOfMyService.MyServiceClassName, $SharePoint.Project.AssemblyFullName$"
    Factory="NameSpaceOfMyServiceHostFactory.MyServiceHostFactoryClassName, $SharePoint.Project.AssemblyFullName$" %>



### Enabling the service help page

### Increasing the upload limit
    
