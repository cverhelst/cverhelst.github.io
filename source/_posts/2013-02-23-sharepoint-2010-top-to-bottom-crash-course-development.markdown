---
layout: post
title: "SharePoint 2010 - Top to Bottom Crash Course - Development"
date: 2013-02-23 00:05
comments: true
categories: 
- sharepoint
- guide
---

SharePoint provides a lot out of the box but at one point you will want to add some custom functionality.

First, let's give you an overview of what "SharePoint solutions" are.
A sharepoint solution is a Visual Studio project containing code that provides the custom functionality that you want to add to your SharePoint environment.
The deployed objects can be all the SharePoint artifacts you can think off:

- Fields
- Content Types
- Lists
- Document libraries
- Views
- Webs
- Sites
- And even custom objects or files.
	* JQuery library

These solutions will contain code but also some special SharePoint XML called CAML. CAML is used for a lot of things in SharePoint development. It can describe how artifacts need to be deployed, how content types are made up. But also where a certain button needs to be placed in the SharePoint UI and what code runs behind it.
Most of these things can be done from code as well (like creating content types) but most of the time developers will use CAML for these kind of things. 

CAML has some benefits over code:

* Takes less time to do it in CAML than in code
* Recommended way

but also some downsides, of which some of them are pretty big:

* It's a one off, meaning, your Fields & Content Types declarations are fine for your first time deploy, but if you need to make changes, you'll have to do it from code because CAML changes are not picked up / they are not supported.
* CAML can be difficult to create from scratch because of the poor IntelliSense.
* CAML is hard to debug, sometimes an XML comment is enough to make it bug out and it's very difficult to find these kind of bugs.

This would probably make you think "Let's just do everything from code", which I know some people have, but that has it's own kind of problems:

* There's no immediate benefit to the "Can't edit the CAML after the first deploy" problem. You'll still need to write additional code to make changes.
* It'll probably be more work to create everything from code.

Basically what this means is, SharePoint lacks [a decent framework](http://spgenesis.codeplex.com) to handle these kind of changes :).

Now, how do you actually "Deploy" your code and artifacts ? Well, SharePoint solutions can contain things called "Features", these are small contains for configuration and code. Moreover, they can be deployed at any level of the SharePoint hierarchy:

* Farm
* WebApplication
* Site
* Web

called the scope. This will be obvious for some cases and less obvious for others when deciding which scope to use for your feature. But to give you an idea:

* Fields & Content Types
	- Site
* Lists
	- Web
* Timer Jobs
	- Web Application

It all depends on the "Scope" of your code / configuration. Fields and ContentTypes can exist at 3 levels:

* Site
* Web
* List

You can't deploy them straight to a List, because that's actually where they are really "Instantiated", i.e. where they are used instead of declared. And guidelines tell you to keep them at the Root of a SiteCollection, i.e. a Site.

So how does this work? Well, you have the SharePoint solution, these contain custom code or elements you want to deploy inside Features. When you Deploy your solution, it makes these features available, but to actually be able to use it or to really create the elements declared in CAML, you need to activate the feature.



