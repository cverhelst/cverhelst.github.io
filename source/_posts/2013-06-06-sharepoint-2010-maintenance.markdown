---
layout: post
title: "SharePoint 2010 - Maintenance"
date: 2013-06-06 22:14
comments: true
categories: 
- sharepoint
---

How do you handle changes to your installed SharePoint environment ? What approach do you take to deploying these changes ? How do you best implement them ? What pitfalls may you encounter ?
All of these questions I hope to answer here, to guide any SharePoint developer facing these challenges.

# First deploy

Your server is set up, your environment is installed along with your solutions and the application is ready to go.

This is the base you will be maintaining, with bug fixes, change requests or just simple improvements. Maybe you're just adding features that are in the pipeline already.

# Changes

There are several different ways to look at the changes you can make to your SharePoint environment, if you categorize them by how they are picked up by the system or what part of the SharePoint environment they affect, ie. their impact.

## Impact

The impact of a change to the existing environment will prove to be a good factor of assessing the risk in deploying the release. It will also be a necessary to know the impact so you can decide on how to implement the change.

Let's start with the beginning.

SharePoint is big and you are allowed to do things in several ways. Some things only have one way for you to do it. In the end it comes down to this:

* CAML (declaratively through XML)
* Code

CAML is unfortunately the trickiest part. Some of it is picked up as you go, some of it you have to _tell_ the system to __update__.
What happens when you delete a fieldLink from a content types' elements.xml ? Does it get applied to your SharePoint environment right away ?
The answer here is: No, it doesn't get applied to the SharePoint environment automatically. In fact, [you shouldn't even be making any changes to it after first release](http://msdn.microsoft.com/en-us/library/aa543504(v=office.14%29.aspx#sectionToggle1). 

Code on the other hand is a lot easier to deploy. You do an Update-SPSolution on your new WSP and SharePoint will use the new dll and therefor the new code. There are only a few exceptions to this, Timer Jobs are one of them. They require that they be reinstantiated before any of the updated Timer Jobs will use the new code. Usually, reactivating the feature that deploys the Timer Job is enough.

Now that we've established that some changes will be recognized automatically and some will not, we can look into the reason for this. When you look closer, you'll notice that all the things that cause you trouble in making SharePoint recognize your changes are nearly always because the way they live in the SharePoint environment is in the form of an Instance. They are living objects.

* Timer Jobs
* Content Types
* Fields
* Sites
* Webs
* Lists
* Views
* WebParts

All of these items are examples of SharePoint artifacts that are instantiated when used in the SharePoint environment and they have to be either modified or recreated before any changes you've made will be manifested. This is what I'd like to call __Provisional Changes__.

### Provisional Change

Provisional Changes are the clumsiest. You will have to write code to make this specific change. This can be either CAML or actual C# code. Either way, it's overhead.

Why do I call it overhead ? Well, CAML allows you to declaratively deploy Content Types, Fields and so on. But it doesn't allow you to make a change in this same CAML that SharePoint will apply to the existing artifacts.

Some things you can alter through [Feature Upgrade](http://msdn.microsoft.com/en-us/library/ee537575(v=office.14%29.aspx)'s CAML, others you'll have to write additional C# code that you will trigger through the Feature Upgrade CustomUpgradeAction.

After you've made a dozen or so of these changes to your SharePoint Solution you'll start to see the difficulty in maintaining this. If you want the CAML way for the initial deploy, you'll have a big list describing all your fields and your content types and their properties (which field is required, which field is added to which content type, which field is shown in which form). When you start having changes, implemented through Feature Upgrades, you'll have to make a "merged" view of these CAML files and any changes you made in code before you can see the actual value of each property of each artifact. 

One guy I know of seems to feel the same way and he made a project called [SPGenesis](http://spgenesis.codeplex.com) that will allow you to manage Fields, Content Types and even List Instances all from their own code file, providing you with only one location where you have to make all your adjustments. Use at your own risk, though.

Another reason this doesn't come off as very maintainable is because, basically, you now have __state__. Your SharePoint artifacts and, generally, your environment, goes from one state to another. That's why you have the versioning in your Feature Upgrades, to determine what state your SharePoint Feature is in and how to go from that state to the latest. This is the crucial part. Sometimes it matters what state you were in __before__ you go to the latest state. Usually, you'll mind for sanity's sake, why make a field required when it was already required, right? Each release will bring with it a new state.
How many states will be live at the same time ? Ideally, only 2. The latest, which is what your developers are working on. And the latest _released_ state, which is what is deployed in production. You will keep adding code to move your production state to the latest with each release.

Most changes to deployed entities or SharePoint artifacts will be of this nature. WebParts, Lists, Views, Webs, Content Types, Fields.

Some of these can be unburdened of the consequences state brings along. WebParts for instance. If you plan ahead, and put them in their seperate feature where you put all the code that builds and adds a certain webpart to a certain page, than you're set. Than you only need some code that removes the webpart again on feature deactivation and you're there. Since WebParts almost never contain any content themselves, but merely provide functionality or content to the viewer, they can be discarded and rebuild without any problems. This basically does away with the state that Content Types and such are in.
When a change needs to happen to this WebPart, you update the code that constructs it and you re-activate the feature. The WebPart is now deployed with the latest changes, no matter what state it was in before. Views are another example. They are essentially stateless, so do yourself a favor and treat them that way :).

### Functional Change

The provisioning change stands very much in contrast to the __functional__ change. Arguably the easiest one to deal with. What I mean with a functional change is merely any change in functionality that is stateless, like a function. You have code that calculates some number from several other input fields, but now needs to change the format it presents it in ? Functional change. You adapt the code, you update the deployed solution and the change is immediately visible afterwards. It is picked up automatically by SharePoint. No overhead.

Some CAML things can also be like this. For example, adding custom actions to a list item's menu through CAML. Let's say you have a typo in the title of the Modal Dialog that's shown when invoking the custom action. No problem, update the CAML (you declare the title to be used in the JavaScript function) and update the deployed solution and done.

# Implementing changes

Now that we have established what impact each kind of change has on your environment, let's look at the options available to you for implementing them.

* CAML
    * Updating an existing artifact (limited)
* Code
    * Updating an existing artifact (full)
    
This doesn't really tell us anything new. CAML is cool for the first time, and than you'll wonder why you didn't use a code approach in the first place :). 

This list doesn't even sum up all the options accurately. We left out PowerShell!

So essentially, you'll be using any of these approaches:

* CAML
    * Feature Upgrade
* Code
    * Add a Feature
    * Feature Upgrade
* PowerShell

Notice you can just add a feature that will run code that makes changes to existing artifacts. I wouldn't advise this unless its for the sake of reducing _state_, like with the WebParts I mentioned, although, you'd want to have it set up like that from the start.

Also notice that these approaches only apply to __Provisional Changes__, functional ones only need a mere update to the existing code base. Unless you're dealing with the inner workings of a custom WebPart or Timer Job which you'll need to somehow redeploy.

## What to choose?

* Adding a new module does get picked up by SharePoint but most likely not provisioned, which means it's not a complete solution in itself. You'll still need some Feature Upgrade CAML on an existing feature, or an entirely new feature to deploy the module. For me, this depends on whether the artifact is related to anything existing or whether it deserves a completely new feature.
* Use feature upgrade CAML for the _simpler_ things
    * Adding fields
    * Add a file
    * Maybe even to add fields to a content type or remove some
* Use feature upgrade Code for the complexer stuff
    * Reconfiguring certain field and content type properties or the order of a field in a content type, etc.
* Use PowerShell when it seems more convenient
    * Uploading files
    * Configuration changes to the highest level site collection
    * Making changes to very specific artifacts
    * To make unforeseen changes, stuff that went wrong and needs to be fixed in this specific instance but will likely not happen again in other deploys
    * But do keep in mind that it will now be part of your Upgrade Process.
    * IMPORTANT: To automatically perform the solution updates and the feature upgrades. (automate as much as you can)

In the case of PowerShell, you can merely ask yourself the question, Is this something I might otherwise do manually ? If no, definitely do it in Feature Upgrade code instead of PowerShell, otherwise you have a good point for doing it in PowerShell.

## Feature Upgrades

Why do you need Feature Upgrades ? Well, not all situations allow you to just merely switch off an existing feature and turn it back on again. Think of all the provisioned artifacts that already exist. You can't just recreate all of them. Features that deploy Lists for example, you certainly don't want to lose the content of your list. Ditto with Content Types. If some particular code created an artifact that you cannot throw away and recreate, you'll have to use a Feature Upgrade. This is true for almost all __Provisioning Changes__.

So that's why you have Feature Upgrades. These will allow you to upgrade existing features, and the artifacts it provisioned.

Only for the very specific changes, you'll consider using PowerShell instead.

Feature Upgrades have the benefit of having the access to your existing code base, so you can reuse functions. On the other hand, you have the limitation and the associated risk of only being able to go through a particular upgrade action only once. Upgrading a feature decisively makes that feature the latest version, there's no upgrading twice if you made a mistake where you would have a need for the feature upgrade to run again. This is in stark contrast to using a Feature or PowerShell.

### Versions

Another possible pitfall of Feature Upgrades are its versions.

You really have to be sure on how you want to be using the __BeginVersion__ and __EndVersion__ attributes of a particular Upgrade Action for a Feature.

Let's summarize how it works:

* A Feature has a version
* A Feature can have multiple Version Ranges with multiple UpgradeActions associated to each Version Range element.
* Each Version Range has a BeginVersion and an EndVersion indicating on which Feature Version it will run the Upgrade Actions.
* BeginVerion is inclusive and EndVersion is Exclusive
    * If a Feature is deployed at version 1, and the latest version is 2
    * and it has a VersionRange element with BeginVersion 1.0.0.0 and an EndVersion of 1.8.0.0
    * it will be upgraded because the VersionRange matches any Feature with version 1.0.0.0 up and             including 1.7.9.9
* If a Feature is detected to have a newer version than the deployed Feature, it will go through the Upgrade Process only once, leaving the deployed Feature in the latest version

This last point has an important implication on the use of your BeginVersion and EndVersion attributes and which WSP you deploy containing which version of that particular Feature. 

Imagine you have an UpgradeAction called _AddFieldXToContentTypeY_ with BeginVersion 0.0.0.0 and EndVersion 1.0.0.0 and the Feature is now versioned at 1.0.0.0.

Your deployed Feature is at version 0.0.0.0 (which is the default when it isn't specified). You deploy a WSP containing that same Feature with version 1.0.0.0. You perform a Feature Upgrade and your feature is now at Version 1.0.0.0 and the UpgradeAction was applied, the field is added to the content type.

In your next change, you add an UpgradeAction called _MoveFieldXToPosition1InContentTypeY_ with BeginVersion 1.0.0.0 and EndVersion 2.0.0.0 and the Feature is now versioned at 2.0.0.0.

You do the same as before, deploying the WSP containing the Feature at version 2.0.0.0 and you perform the Feature Upgrade. Cool, your deployed Feature now got upgrade to version 2.0.0.0 and the field was moved to its new position in the order of fields in the content type.

All well and good but what happens when you do a clean install of your WSP containing the original Feature definiton with version 0.0.0.0 and you deploy the WSP containing the Feature of version 2.0.0.0 and you upgrade this feature ? Ha! You'll see that field X got added to the content type but it won't be in position 1 of the ordering of the fields of the content type. Why's that ? Because the deployed Feature of version 0.0.0.0 did not match the VersionRange of the second UpgradeAction of 1.0.0.0 to 2.0.0.0. 

This is obvious now, but when you're deciding on the Version Ranges it may seem more obvious of matching BeginVersion to previous upgrades EndVersions, no ? After all, you're going from version 0 to version 1 to version 2, right ? Well, this is wholy up to you, do you want to be forced to go through each version deploy or not ? It may seem like an easy choice in this case, only one feature to upgrade and no dependencies, but when you have Feature Upgrades spread out over 3 or more Features that you need to upgrade in a certain order, maybe even execute a few PowerShell scripts in between, you won't be so keen in allowing your environment to go from version 0 to version 2 in one go. What's more, after you've done the Feature Upgrade, that Feature is version 2.0.0.0, no matter if it performed that second UpgradeAction or not. You won't even be able to execute it without doing a redeploying where you manipulate the versions again.

I would advise to make BeginVersion 0.0.0.0 as a default and only in very rare cases change it to a specific version. Likewise, make the EndVersion match a generic release version, don't make it too specific.

## Deploy and Upgrade scripts

Remember when we talked about the state of a SharePoint environment earlier ? Ideally you'll only want 2 states, the deployed state and the latest developed state.

This would indicate you don't want to know about any past versions that might have existed, you just want to be able to go immediately to your latest deployed state and start working on upgrading it to the latest developed state. 

You must realise that this is a very "ideal-world" way of thinking. This can work perfectly if you only have your WSP to think about, and the Features inside it. When you have to calculate in the PowerShell scripts that are needed to upgrade to go from one version to another, you're in a whole different situation. 

When I say PowerShell scripts, I don't mean the script you use to deploy the WSP's with Update-SPSolution and trigger the Feature Upgrades / Install and activate any new features. That's perfectly fine. What I'm talking about is the scripts you use to make these very granular changes, like setting Web Properties, Web Application Properties, re-activating Web Application Features (when dealing with Timer Job code upgrades this is necessary), perhaps adding a search center and configuring search in your application. These kind of changes are more difficult to think of as mere additions to your "clean install" and less so as an upgrade from one state to another.

_Scenario_: you have a version 1 SharePoint environment deployed. You make a large impact change (for exmaple, any of the things I mentioned in the previous paragraph) to get it to version 2. You want to stay in the "Ideal-World" where you only have 2 states to worry about (don't forget about version 3 that's on it's way, making it a total of 3 possible states now). What you'll have to do is the following:

* Make an upgrade scenario, where you call any of these new scripts that perform the big impact changes
* Integrate this scripts back into the "clean install" scenario (let's call this state version 0)

Now you can safely go from version 0 (nothing) to version 2 or from version 1 to version 2.

When deploying the changes of version 3 you have to worry about upgrading from both a version 1 or version 2 environment (don't think of this as impossible, there's multiple reason you might want or need to have an older version environment around) as well as a clean install from version 0. 

* You'll have to add another upgrade scenario script
* Update the clean install script with these new changes

Cool, you've managed to stay in your "ideal-world". But can you really trust this "clean-install" scenario compared to the state your production environment is in ? After all, this one went from version 0 to version 1, upgraded to version 2 and later to version 3. Whereas your clean install will go from version 0 to version 3 straight up. But what about the WSP's!? Well, in this case (large impact changes mandating the need of special PowerShell scripts) you'll have to keep each WSP of each version around and perform an Update-SPSolution between each version change.

Your clean install script will look something like this over the course of these upgrades:

#### First deploy

    Install.ps1
    
        // Deploy version 1
        Update-SPSolution -Identity "R1\MySolution.wsp" -LiteralPath (gci "R1\MySolution.wsp")
        .\CustomConfigurationForVersion1.ps1

#### Upgrade to version 2

    Install.ps1

        // Deploy version 1
        Update-SPSolution -Identity "R1\MySolution.wsp" -LiteralPath (gci "R1\MySolution.wsp")
        .\CustomConfigurationForVersion1.ps1

    UpgradeToVersion2.ps1
    
        Update-SPSolution -Identity "R2\MySolution.wsp" -LiteralPath (gci "R2\MySolution.wsp")
        .\FeatureUpgradesForVersion2.ps1
        .\CustomConfigurationForVersion2.ps1
    
#### Upgrade to version 3

    Install.ps1
    
        // Deploy version 1
        Update-SPSolution -Identity "R1\MySolution.wsp" -LiteralPath (gci "R1\MySolution.wsp")
        .\CustomConfigurationForVersion1.ps1
        
        // Upgrade to version 2
        .\UpgradeToVersion2.ps1
        
    UpgradeToVersion2.ps1
    
        Update-SPSolution -Identity "R2\MySolution.wsp" -LiteralPath (gci "R2\MySolution.wsp")
        .\FeatureUpgradesForVersion2.ps1
        .\CustomConfigurationForVersion2.ps1
    
    UpgradeToVersion3.ps1
        
        Update-SPSolution -Identity "R3\MySolution.wsp" -LiteralPath (gci "R3\MySolution.wsp")
        .\FeatureUpgradesForVersion3.ps1
        .\CustomConfigurationForVersion3.ps1
        
Like I said, you have to keep around the WSP's of previous version because of the Feature Upgrade where you might need to do some PowerShell stuff in between, or need to follow a specific order of upgrading (Feature X to version 2, Feature Y to version 2 before upgrading Feature X to version 3, I dunno man, this stuff happens more quickly than you'd think).

Keeping the WSP's around and upgrading from one version to another following all the in between steps gives you the exact same upgrade process as your deployed production environment, which is always what you should aim to be developing on. Not some shortcut deployed environment that doesn't have the same history as your production environment. You'll be sorry when you upgrade your production environment and discover it suddenly behaves differently than your development environment.

Even better, develop on restores from production backups.
 
## Feature Instances

Another important fact to keep in mind is that Features have instances. They come forth based on their scope and if you have a Site Collection with some Site Features and you have 10 SubWebs with some Web Features you'll have 1 instance of each Site Feature (on the Site Collection) and 10 instances of each Web Feature (on all the SubWebs). This matters for your Feature Upgrade code as well, this is basically the reach they have controls over. If you have a Web Feature that deploys a List in a Web, and you have 10 Webs with this Feature activated, you'll have 10 instances of the Feature that deployed this List to each Web, and you'll have 10 Feature Upgrades to execute, albeit with the identical Feature Upgrade code.

I can highly recommend using the Feature Upgrade Kit from Chris O'Brien, but if you need a tighter control over the order of upgrading the Features, you'll want to script it yourself. This is pretty easy as you can just query the Site object for any features that need to be upgraded (it will return features from subwebs as well).

    $Site = Get-SPSite("http://mysite")
    $FeatureScope = [Microsoft.SharePoint.SPFeatureScope]::Site
    $OnlyRequiringFeatureUpgrade = $true
    $FeaturesRequiringUpgrade = $Site.QueryFeatures($FeatureScope ,$OnlyRequiringUpgrade)
    
Here it becomes important to remember you have an instance of a Feature for each Site or Web it is deployed on:

    $ForceUpgrade = $false
    $FeaturesRequiringUpgrade | % { 
        $_.Upgrade($ForceUpgrade)
    }
    
I prefer to be absolutely certain and create a list of Feature names I want to see upgraded, query the Site for Features requiring an upgrade, filter out the ones I want to upgrade, remember each Feature's current version, perform the upgrade and compare the versions, if they're still the same I will give feedback about this. Ditto in case the feature I wanted to upgrade cannot actually be upgraded (because it already is at it's latest version, or so SharePoint thinks).

This approach is particulary useful when you need tight control over the order of Feature Upgrading. This might matter when Features of different scope need to be upgraded.

## Feature Activate & Feature Upgrade Events

If you're required to make __Provisioning Changes__ where you'll have to implement them through the use of Feature Upgrades to upgrade any existing artifacts, you'll have to remember to not forget about how the new artifacts have to be created.

This is why it's advised to seperated the code making the specific change out of the Feature Upgrade event and also put a call to this code into the Feature Activate event. This way, it's just as if the existing Feature Instances ran the same code as the latest Feature Activate events are and they stay in perfect sync, which is the whole point after all.

This brings us to the next point I wanted to touch on.

# Artifacts -  New vs. Existing

After you've successfully deployed your changes to the SharePoint environment, there'll be a distinct difference between the SharePoint artifacts to keep in mind. Those that already existed, and those that have been created after the deploy.

This may not seem that important of a difference, but when you've made mistakes and are seeing strange things happen in your SharePoint environment, be sure to check which of the artifacts are affected, are the old artifacts experiencing issues or is it happening with the newly created ones ? This way, at least, you'll know where to look. Clear indication of inconsistencies in code triggered by the Feature Upgrades and the Feature Activate events.

# Summary

To sum it all up, if you stick to the following, you may just be fine :):

* Be aware of the nature of a change, provisional or functional ?
* Make clear distinction between the install/upgrade script of each version & the WSP's / other files needed for each version. Keep integrating each new version back into the "clean install" script.
* Be wary of any differences between the code that execute for creating new artifacts and the code that executes on existing artifacts to bring them up to speed.
* Give preference to code contained in the WSP over added PowerShell scripts, and be aware of the nature of Feature Upgrades versus adding a new Feature (run just once vs being able to run again).
* Make BeginVersion in FeatureUpgrades be 0.0.0.0 as a default and only in very rare cases change it to a specific version. Make the EndVersion match the generic release version, don't make it too specific.

# Miscellaneous

* I prefer Update-SPSolution over the complete Retract/Remove/Add/Deploy cycle. Features get reactivated in the latter and that scares me. I sure know my VS Retract Features breaks the whole environment...
* When using just Update-SPSolution to upgrade the WSP, remember to run Install-SPFeature on any newly created Features in your solution. It merely "installs" the Feature in the farm so you can activate it where you like. This is something that doesn't automatically happen when using Update-SPSolution opposed to the full Retract/Remove/Add/Deploy cycle.
* When you have issues with locked .dll's because they're still loaded in your PowerShell Shell (so annoying) be aware that you can launch new Shells inside your existing Shell. These Shells will be starting afresh and won't have old dll's loaded. Be aware that you'll have to close this Shell as well, using either exit, or the following approach, which will close the shell right after the last line in the CodeBlock is executed.
    
        PowerShell -Command {
            Some powershell here
        }
* Script everyting :)