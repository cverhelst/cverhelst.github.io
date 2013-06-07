---
layout: post
title: "SharePoint 2010 - Maintenance"
date: 2013-06-06 22:14
comments: true
categories: 
- sharepoint
---

How do you handle changes to your installed SharePoint environment ? What approach do you take to deploying these changes ? How do you best implement them ? What is even possible ?
All of these questions I hope to answer here, to guide any SharePoint developer facing these challenges.

# SharePoint 2010 - First deploy

Your server is set up, your environment is installed along with your solutions and the application is ready to go.

This is the base you will be maintaining, with bug fixes, change requests or just plain improvements. Maybe you're just adding features that are in the pipeline already.

# SharePoint 2010 - Changes

There are several different ways to look at the changes you can make to your SharePoint environment, if you categorize them by how they are picked up by the system or what part of the SharePoint environment they affect, ie. their impact.

## Impact

The impact of a change to the existing environment will prove to be a good factor of assessing the risk in deploying the release. 

### Provisioning Change

Let's start with the beginning.

SharePoint is big and you are allowed to do things in several ways. Some things only have one way for you to do it. In the end it comes down to this:

* CAML (declaratively through XML)
* Code

CAML is unfortunately the trickiest part. Some of it is picked up as you go, some of it you have to _tell_ the system to __update__.
What happens when you delete a fieldLink from a content types' elements.xml ? Does it get applied to your SharePoint environment right away ?
The answer here is: No, it doesn't get applied to the SharePoint environment automatically. In fact, [you shouldn't even be making any changes to it after first release](http://msdn.microsoft.com/en-us/library/aa543504(v=office.14%29.aspx#sectionToggle1).

This is what I'd call a __provisioning__ change.

Because of this, you can't merely reconfigure the content type. They need to be altered another way. Some things you can do through [Feature Upgrade](http://www.sharepointnutsandbolts.com/2010/06/feature-upgrade-part-1-fundamentals.html)'s CAML, others you also have to do through Feature Upgrades (or new features) but it requires you to write extra code, specifically for this particular change you want to make.

This doesn't come off as very maintainable. This is because basically you now have _state_. Your SharePoint environment goes from one state to another. That's why you have the versioning in your Feature Upgrades, to determine what state your SharePoint environment is in and how to go from that state to the latest. This is the crucial part. Sometimes it matters what state you were in __before__ you go to the latest state. Usually, you'll mind for sanity's sake, why make a field required when it was already required, right? Each release will bring with it a new state.
How many states will be live at the same time ? Ideally, only 2. The latest, which is what your developers are working on. And the latest _released_ state, which is what is deployed in production. You will keep adding code to move your production state to the latest with each release.

Most changes to deployed entities or SharePoint artifacts will be of this nature. WebParts, Lists, Views, Webs, Content Types, Fields.

Some of these can be unburdened of the consequences state brings along. WebParts for instance. If you plan ahead, and put them in their seperate feature where you put all the code that builds and adds a certain webpart to a certain page, than you're set. Than you only need some code that removes the webpart again on feature deactivation and you're there. Since WebParts almost never contain any content themselves, but merely provide functionality or content to the viewer, they can be discarded and rebuild without any problems. This basically does away with the state that Content Types and such are in.
When a change needs to happen to this WebPart, you update the code that constructs it and you re-activate the feature. The WebPart is now deployed with the latest changes, no matter what state it was in before. Views are another example. They are essentially stateless, so do yourself a favor and treat them that way :).

### Functional Change

The provisioning change stands very much in contrast to the __functional__ change. Arguably the easiest one to deal with. What I mean with a functional change is merely any change in functionality that is stateless, like a function. You have code that calculates some number from several other input fields, but now needs to change the format it presents it in ? Functional change. You adapt the code, you update the deployed solution and the change is immediately visible afterwards. It is picked up automatically by SharePoint. No overhead.

Some CAML things can also be like this. For example, adding custom actions to a list item's menu through CAML. Let's say you have a typo in the title of the Modal Dialog that's shown when invoking the custom action. No problem, update the CAML (you declare the title to be used in the JavaScript function) and update the deployed solution and done.

# SharePoint 2010 - Changes implementation

Now that we have established what impact each kind of change has on your environment, let's look at the options available to you for implementing them. I've added (full) and (partial) tags to indicate what is possible with each approach.

* CAML
    * Adding a new artifact (full)
    * Updating an existing artifact (partial)
* Code
    * Adding a new artifact (full)
    * Updating an existing artifact (full)
    
This doesn't really tell us anything new. CAML is cool for the first time, and than you'll wonder why you didn't use a code approach in the first place :). You'll notice I added (full) and (partial) at the end, but even CAML for new items isn't really full featured because you simple cannot do everything with CAML, so the (partial) tag really only is relative to what you could do __if__ you were working with a new artifact.

This list doesn't even sum up all the options accurately. We left out PowerShell!

So essentially, you'll be using any of these approaches:

* CAML
    * Add a Module
    * Feature Upgrade
* Code
    * Add a Feature
    * Feature Upgrade
* PowerShell

Notice you can just add a feature that will run code that makes changes to existing artifacts. I wouldn't advise this unless its for the sake of reducing _state_, like with the WebParts I mentioned, although, you'd want to have it set up like that from the start.

Also notice that these approaches only apply to Provisional Changes, functional ones only need a mere code update. Unless you're dealing with the inner workings of a custom WebPart which you'll need to somehow redeploy.

* Adding a new module does get picked up by SharePoint but most likely not provisioned, which means it's not a complete solution in itself. You'll still need some Feature Upgrade CAML on an existing feature, or an entirely new feature to deploy the module. For me, this depends on whether the artifact is related to anything existing or whether it deserves a completely new feature.
* Use feature upgrade CAML for the _simpler_ things
    * Adding fields
    * Add a file
    * Maybe even to add fields to a content type or remove some
* Use feature upgrade Code for the complexer stuff
    * Reconfiguring certain field and content type properties
        * The other of a field in a content type, etc.
* Use PowerShell when it seems more convenient
    * Uploading files
    * Configuration changes to the highest level site collection
    * Make changes to very specific artifacts
    * To Make unforeseen changes, stuff that went wrong and needs to be fixed in this specific instance but will likely not happen again in other deploys
    * But do keep in mind that it will now be part of your Upgrade Process.
    * IMPORTANT: To automatically perform the solution updates and the feature upgrades. (automate as much as you can)

In the case of PowerShell, you can merely ask yourself the question, Is this something I might otherwise do manually ? If no, definitely do it in Feature Upgrade code instead of PowerShell, otherwise you have a good point for doing it in PowerShell.

   