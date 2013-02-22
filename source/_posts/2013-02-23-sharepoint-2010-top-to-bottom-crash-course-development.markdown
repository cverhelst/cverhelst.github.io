---
layout: post
title: "SharePoint 2010 - Top to Bottom Crash Course - Development"
date: 2013-02-23 00:05
comments: true
published: false
categories: 
- sharepoint
- guide
---

SharePoint provides a lot out of the box but at one point you will want to add some custom functionality.
I'll get into the required tools and setup in a second, but first, let's give you an overview of what "SharePoint solutions" are.
A sharepoint solution is a Visual Studio project containing code that provides custom functionality, or at the very least automates some configuration stuff for you like deploying SharePoint objects).
The deployed objects can be all the built-in SharePoint building blocks you can think off:

- Fields
- Content Types
- Lists
- Document libraries
- Views
- Webs
- Sites
- And even custom objects or files.

These solutions usually contain code but nearly always they'll contain CAML as well. It's SharePoint's XML to 'deploy' these built-in SharePoint objects I mentioned earlier. Now, it's certainly possible to create them from code as well, but usually it is preferred to declare them in CAML and go from there. 
Most of the time your actual code will merely be logic that takes care of the custom functionality you want to provide.

Say you want to be able to move an item from one list to an other, you would do it with custom code deployed from a SharePoint solution, that adds a button somewhere that allows the user to move an item to another list. How it works is entirely up to you to decide.
But if you wanted to add content types with fields to a certain site and there would automatically be a list using that Content Type, this would be a perfect job for CAML. Still using a SharePoint solution to deploy this 'Feature', but not from code but xml instead, declaritively, as it's called.

Why did I say 'Feature'? Because automatically deploying Content Types and a list or that copy-an-item stuff we saw earlier are features, as in, they're functionality. But in SharePoint you also have a thing called features, as in, little containers of functionality or elements you want to deploy.
So 

So how does this work? Well, you have the SharePoint solution, these contain custom code or elements you want to deploy inside Features. When you Deploy your solution, it makes these features available, but to actually be able to use it or to really create the elements declared in CAML, you need to activate the feature.
Footnotes
FEATURES
Features can be deployed at a different level, or rather they can exist at a certain predefined level. Depending on the scope your code or elements are relevant to, you will want to create them to be one of the following:
List Feature
Web Feature
Site Feature
Web Application Feature
Farm Feature
I guess it's pretty obvious how this works. It's interesting to know that if you have activated a Web Feature at several webs, there are actually multiple instances of that feature active. But you will never have to worry about this until you get to need Feature upgrading.
CAML 
You're probably wondering why you would need two ways to create or deploy files or objects ( being CAML and code ). When you start developing for SharePoint you'll notice things need to be done in a certain way because that's how SharePoint works.
In the case of CAML vs code, you'll generally want to use CAML in the following cases:
Your elements are static in nature, meaning you'll expect to be the way you  created them when you access them later. This has a lot to do with the fact SharePoint uses Guids as IDs for object, allowing you to reliably get at the object. Guids can only be choosen or assigned through a CAML declaration, not in code.





