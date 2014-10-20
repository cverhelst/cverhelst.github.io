---
layout: post
title: "SharePoint - Hosting a WCF service with a SPContext"
date: 2014-10-19 15:07
comments: true
categories: 
- sharepoint
- wcf
- service host factory
- spcontext
---

If you want to extend SharePoint, adding custom web services is one way to do it. There's several requirements usually, 
and if one of them is the necessity for a SPContext, you can use the built in [SharePoint WCF Service Factories](http://msdn.microsoft.com/en-us/library/office/ff521586.aspx).

There's a couple of things to keep in mind when using Service Host Factories. They're basically the programmatic alternative to the web.config configuration files. This has the benefit of not having to deploy or update a web.config file somewhere, but at the same time it has the downside of not even being able to quickly reconfigure something by putting a web.config next to the .svc file. It's all code from here, no configuration.

## Deployment

These .svc files can be deployed to the ISAPI hive folder, in their own subfolder preferably. They'll be accessible by this url from any site collection in the farm:

- http://{sitecollectionurl}/{subweburl}/_vti_bin/{pathToMySvcFileInTheISAPIFolder}/{myservicefilename}.svc
- _vti_bin points to the ISAPI folder, so {pathToMySvcFileInTheISAPIFolder} would be the hierarchy of folders you used inside the ISAPI folder before you see your {myservicefilename}.svc file.

It is "web" aware, so your SPContext.Current.Web will be whatever web you called the service at. Keep this in mind if you want to locate artifacts in your web / sites, because this would mean you'll have to call your service from the correct web to make them accessible.

Because the .svc file is deployed to the HIVE, you will need a SharePoint project in your solution to deploy the file with. This makes deployment itself really easy, you don't even have to activate a feature or create an IIS site. SharePoint takes care of everything.

## Creating the service

A service factory is specified in the .svc file. SharePoint provides 3 service factories for you to use, depending on the kind of service you want to create:

- SOAP: MultipleBaseAddressBasicHttpBindingServiceHostFactory
- REST: MultipleBaseAddressWebServiceHostFactory
- ADO.NET Data Service : MultipleBaseAddressDataServiceHostFactory

I wanted to make a REST service, so I used the MultipleBaseAddressWebServiceHostFactory in my case. This means my .svc file looked like this: 

```xml
    <%@ServiceHost Language="C#" Debug="true"
    Service="NameSpaceOfMyService.MyServiceClassName, $SharePoint.Project.AssemblyFullName$"
    Factory="Microsoft.SharePoint.Client.Services.MultipleBaseAddressWebServiceHostFactory, Microsoft.SharePoint.Client.ServerRuntime, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" %>
```
You'll notice we're using a VS Token in this file, to configure VS to replace the token in .svc files you can [follow the instructions described on MSDN](http://msdn.microsoft.com/en-us/library/office/ff521581.aspx#code-snippet-4).
    
For it to work correctly, you'll have to add [some extra attributes](http://msdn.microsoft.com/en-us/library/office/ff521581.aspx#code-snippet-2) to the class implementation of your service:

- BasicHttpBindingServiceMetadataExchangeEndpointAttribute
- AspNetCompatibilityRequirements(RequirementsMode = AspNetCompatibilityRequirementsMode.Required)

Additionally, you can add the following attribute if you want to have the exception details shown when browsing to the service:

- ServiceBehavior(IncludeExceptionDetailsInFaults=true)

Once that's done you can start adding service contracts & service methods the way you normally do in WCF.

### Security

The webservice is callable by anybody. You will not be blocked from the service because you do not have access to the SharePoint site that you called the service from.

You _WILL_ be blocked from interacting with the SharePoint Artifacts at that location if the credentials you used do not have sufficient permissions on those SharePoint Artifacts.

So you're responsible for handling security outside of SharePoint interacting code. This also means any `SPSecurity.RunWithElevatedPrivileges` code blocks as they basically allow anybody to run that piece of code. Even if they're not even known inside SharePoint.

### Service Methods

You'll first have to create an interface with the _[ServiceContract]_ attribute on the class and _[OperationContract]_ attribute on the interface methods.

Next, you can have the .svc code behind class implement this interface

#### Updating SharePoint artifacts

If you want your service methods to expose functionality that makes changes to SharePoint artifacts (create a list, remove a list item, update list item properties) you will run into some issues.

- GET request
  - This will give you errors regarding ["unsafe updates"](http://msdn.microsoft.com/en-us/library/office/gg552614.aspx#code-snippet-9)
    - This is to prevent an unsuspecting user to accidently execute code with unintended consequences (because of a link that was injected somewhere in the page).
- POST action
  - This will give you errors because of the SharePoint Page Form Digest which helps prevent [CSRF (cross site request forgery)](http://msdn.microsoft.com/en-us/library/office/gg552614.aspx#bestpractice_crossrequest)
    - This is to prevent a user's credentials to be used in a different domain than the page the user visited.

The work arounds in these cases is to use `web.AllowUnsafeUpdates = true` and `SPUtility.ValidateFormDigest()`

The truth is that for either of these, in the case of Web Service methods, you're kind of stuck. Sure, in your GET method you can set web.AllowUnsafeUpdates to true, but you don't always control the creation of the SPSite/SPWeb objects being used by the underlying SharePoint code. 

The same goes for the POST method, you can go get a digest token from SharePoint and validate that before calling the webservice, but that's putting an extra burden on the client calling the service. 

In essence, these security measures have been in the context of public facing sites / web services. SharePoint is often used for intranets and in my case, this is where we were developing these services for.

Never trust user input. Even in intranet situations.

#### GET Method

Let's assume your GET method needed to do something, that `web.AllowUnsafeUpdates = true` doesn't help you do. How does SharePoint know it's a GET request ? It first checks to see if there's an SPContext... Yes, the very reason we've built our service like this was to have an SPContext in the first place. And now it's basically the reason we cannot execute our code from inside a GET request.

Use case: Our situation needed to have a GET method that could be called upon a certain trigger to archive a SharePoint artifact. In this case, we could validate the conditions that would mean the artifact had to be archived inside the service method. This means any malicious use of the service could not be used, as it was just a trigger to CHECK if it needed to be archived and to then also archived it, worst case, we were gonna be doin a lot of unnecessary checks if a random person was calling the service method. 

Yet, the artifact we were dealing with was a DocumentSet and the underlying code was recreating an SPSite / SPWeb object so we couldn't use the recommended method of setting `web.AllowUnsafeUpdates = true` .



It's probably a good idea to wrap this in an object implementing IDisposable and saving the context in a back variable so you can put it back after.

#### POST Method

As we saw before, the POST actions get special treatment too by means of the Form Digest token.

This is because it's almost naturally assumed that any POST request in a SharePoint context will happen from a form. Yeah right. There's no such thing like calling WCF services from outside of a web context, like, an office app ? Think again.

Use case: We needed to allow a file to be upload to a predetermined library. We already know that only authorized people can do these things, both in SharePoint and from our service method. That's the only security we were gonna have on the service method. Since we were not particularly concerned for any CSRF inside our intranet, we were not gonna validate a form digest that the client had to request himself explicitly (on web pages it is served along by SharePoint for you), we were gonna disable the check.

This also happens with `web.AllowUnsafeUpdates = true`, for some reason. In our case it was sufficient, but if not we'd had to have to use the same workaround as explained below.

#### Security issues workaround

The workaround was to unset our SPContext, which meant SharePoint would think we were executing from an application rather than an http context, and would no longer block changes to the SharePoint artifacts.

```csharp
    HttpContext.Current = null
```

Keep in mind I don't really approve of this trick. I just don't see a good way that was made available by SharePoint out of the box. For GET requests, the alternative isn't sufficient in some cases, for POST requests, it's just dumb to have to make 2 WCF calls to get the same thing done.

But if you're aware of the risks and make sure you're prepared for any "malicious" service calls, as in, don't make your service methods to any irreversible things in case they we're executed in unintended situations, I believe it's a good alternative.

### Using multiple service contracts within the same service codebehind

This title may not be so clear at first. The idea is to reuse the same listenUri endpoint (base url) to host multiple service methods, that have been defined in _different_ service contracts.

What does this look like in the codebehind of your .svc file (the .svc.cs file) ? Like so:

```csharp
    public class MyService : IMyServiceContractForHR, IMyServiceContractForFinance
    {
    
    }
```

Pretty straight forward. Except....


Yeah, SharePoint ofcourse. If you look in the implementation of (at least) the MultipleBaseAddressWebServiceHost you'll find that it has a method for adding the default endpoints. It even has a property especially for holding all the Service Contracts it detected that were being implemented by the service.

And than it takes the first one, adds endpoints for it and call it a day.

I honestly don't know if there might be a technical reason for this, but if vanilla WCF allows you to do this, than why doesn't a SharePoint specific service host factory allow you to do it ? And why in the frigging **** do they have to make it all `internal` every frigging time. I wasn't gonna bother creating the endpoints as closely to the real thing as possible (I'm no WCF expert believe it or not), which may not even be totally necessary. But you know, this whole service was tested and prepered under it's current configuration. It was meant to implement multiple contracts. It was gonna implemented multiple contracts.

Enter: The interface that inherits from all.

Yeah, apparently this works. Just have a different interface, IMyService, that inherits from all the Service Contracts you want to implement and you're done. 

So, what were we making a fuss about again ? Oh right.

## Creating a custom service host factory

If you're using the SharePoint provided service host factory it means you're using the service as it's configured by SharePoint:

- 3 endpoints
  - /
  - /anon
  - /ntlm
- no /help page to browse to
- default 2mb upload limit

If you want to change these properties of your service, remember, placing a web.config near the .svc file will not make a difference. You'll have to subclass the service host factory and change the endpoints that have been created by SharePoint base class and update their settings in the __OnOpening__ event from the service host.

```csharp
public class CustomMultipleBaseAddressWebServiceHostFactory : Microsoft.SharePoint.Client.Services.MultipleBaseAddressWebServiceHostFactory
    {
        protected override System.ServiceModel.ServiceHost CreateServiceHost(Type serviceType, Uri[] baseAddresses)
        {
            return new CustomMultipleBaseServiceHost(serviceType, baseAddresses);
        }
    }

    public class CustomMultipleBaseServiceHost : Microsoft.SharePoint.Client.Services.MultipleBaseAddressWebServiceHost
    {
        public CustomMultipleBaseServiceHost(Type serviceType, params Uri[] baseAddresses)
            : base(serviceType, baseAddresses)
        {
        }
    }
```

This also means you'll have to change the reference to the service host factory used by your service in the .svc file:

```xml
    <%@ServiceHost Language="C#" Debug="true"
    Service="NameSpaceOfMyService.MyServiceClassName, $SharePoint.Project.AssemblyFullName$"
    Factory="NameSpaceOfMyServiceHostFactory.MyServiceHostFactoryClassName, $SharePoint.Project.AssemblyFullName$" %>
```

### Updating endpoint configuration

The best way to updating the endpoints created by SharePoint by default is to do it in the _OnOpening_ event. I prefer doing this to stay as close as possible to the default SharePoint setup opposed to clearing the list of default endpoints and recreating them manually with your preferred way of configuration.

```csharp
    protected override void OnOpening()
    {
        base.OnOpening();

        foreach (ServiceEndpoint endpoint in this.Description.Endpoints)
        {
            EnableHelpPageOnWebHttpBehavior(endpoint);
            IncreaseFileUploadSize(endpoint);
            
        }
    }
```

You need to call the base method first because that one will take care of creating the default endpoints.

#### Enabling the service help page

```csharp
    private static void EnableHelpPageOnWebHttpBehavior(ServiceEndpoint endpoint)
    {
        foreach (var webHttpBehavior in endpoint.EndpointBehaviors.OfType<WebHttpBehavior>())
        {
            webHttpBehavior.HelpEnabled = true;
        }
    }
```
#### Increasing the upload limit
    
```csharp
    private static void IncreaseFileUploadSize(ServiceEndpoint endpoint) {
        var customBinding = endpoint.Binding as WebHttpBinding;
        if (customBinding != null)
        {
          customBinding.MaxBufferSize = Int32.MaxValue;
          customBinding.MaxReceivedMessageSize = Int32.MaxValue;
        }
    }
```
