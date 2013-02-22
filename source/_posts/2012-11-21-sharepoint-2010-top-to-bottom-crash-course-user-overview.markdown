---
layout: post
title: "sharepoint 2010 - Top to Bottom Crash Course - User Overview"
date: 2012-11-21 20:30
comments: true
published: false
categories: 
- sharepoint
- guide
---

# Introduction

This articles aims to provide a quick overview about what __using SharePoint__ means from a __users__ viewpoint, and tries to explain all its different facets.

# Quick look

## Interface

It's called a _crash_ course for a reason. This is what a SharePoint site looks like: 

![Picture of a SharePoint site](/path/to/image.png "SharePoint Interface")

The image above show what a SharePoint page looks like for a visitor, and now when it's in __Edit Mode__:

![Picture of a SharePoint page in edit mode](/path/to/image.png "SharePoint Page in Edit Mode")

The biggest differences are the ribbon that has shown itself to the user, as well as page showing all kinds of toolbars to allow editing straight into the page.

All this depends on what __page__ you are looking at, more specifically, what _context_ you are currently in. I will explain this later, but for now, it is important to know that this is _context aware_, meaning that the functionality provided depends on the type of page you currently have open, where your cursor has focus and even what permissions you have.

--- Extend this section

## Page Editing Functionality

Now, what functionality is available to a SharePoint user who wants to create content on a page ? Take a look:

### Basic

This is very much like the Office Word experience on a Web Page. 
So on an ordinary web page, users with sufficient permissions can edit the page to:

- Edit a paragraph (very __Office Word__ like)

![Picture showing the text editor](/path/to/image.png "SharePoint Text Editor")

- Insert an image

![Picture showing how to insert an image](/path/to/image.png "Images on SharePoint Pages")

- Insert a hyperlink

![Picture showing how to insert a hyperlink](/path/to/image.png "Hyperlinks on SharePoint Pages")

- Edit Styles

### Advanced

Looking a little further than just the content of the page, SharePoint has some more advanced features available to the users of the platform:

#### Version control

Users have control over the files they're working on. _Versioning_ allows users to 

- Keep old versions of their files.
- Lock a file so only they can edit it at that time
- Work with minor (drafts) and major(complete) versions: They can keep working on a file while the other users only see that last complete version of that file. When they're done making their changes, they can check it back in as a major version and allow the other users to see the changes made.

Very closely related to this is the ability to edit a Word Document directly on the web site. You don't need to download the file, open it in Office Word (which you would need to have installed) and then edit it. Save it, and upload it again. Nope, none of that in SharePoint. You edit the document directly on SharePoint, save it, and that's that.
    
#### Publishing

Closely related to the __versioning_ functionality is the act of publishing a page. This is useful in environments that are public facing, as in, they're on the _internet_. Pages are kept "of the record" until they're _published_, at which point the _public_ can see the page.

#### Wiki

The special Wiki Pages have built-in functionality that help creating a Wiki by allowing users to type links that will immediately create a new page when the creator follows it.

# SharePoint for Users - Top down overview

Now that you've had a quick introduction to what SharePoint has to offer to the users, we will take a few steps back to look at all its features from a more logical starting point: the beginning.

So how is SharePoint structured ? What does this mean for the user ? 

## Structure

SharePoint, for the users, consists of the following components:

- Webs
- Libraries
- Lists
- Pages
    * Web Parts

Technically incorrect (there are Site Collections and Sites, two different things but pretty self explanatory) but functionally for a user, this is it. This is all you need to store content on a SharePoint site. Lets go through them one by one:

### Webs

Webs, as you all probably know, are the container for a site. They have a home page, contain all the other components like lists and libraries and even other webs. They usually contain information that belongs to the same context, that belongs together. For example, a project might have its own Web. A better example will be provided later, linking all the components together.

### Libraries

SharePoint has different types of libraries to store different kind of files, Document Libraries being the most used. But there are also Asset Libraries to store media files such as audio and video. These will contain all sorts of files that need to be stored on SharePoint. For instance, all the web pages on a SharePoint Site (Web) are stored in a Library.

### Lists

Sometimes you don't want to store a whole file, just information, a record, grouping together different fields of information. SharePoint stores this kind of information in Lists. All the people working on a certain project might be in a List, with all the relevant information attached to a record, or item as SharePoint calls it, in the list. First Name, Last Name, Telephone Number and other kinds of relevant information can all be stored in fields of an item in a List. 

Having all this information so neatly together in the same place opens up possibilities to help the users further. Filtering, sorting, grouping the information, it is all possible in SharePoint Lists. If you want to enter the informaton in bulk from an Excell sheet ? No problem. What if you have thousands and thousands of items in the list and you happen to only work on items with field value X or Y ? No problem, create a __View__ that filters the items based on field values for you beforehand, so whenever you look at the list, you only see the information relevant to you.

### Pages

This is a special case. The other components were contains of information, but this is a single object, right ? You're only partially wrong. Much like the List holds items with different fields and information, a page holds content meant to be viewed by other users. They have the unique property of __making up the web__. They can be used to group links that are relevant only to a certain group of users. They could bookmark this page and have they're own __portal__ to get at their content. They can view parts of the content or information by means of _Web Parts_. Web Parts are the building blocks held by the __containing__ page (see, it's a container after all). Web Parts can be added to a page to provide quick overviews, allow extra functionality or help the user in some other way. 

So what can be a Web Part ? Lets say you want to show the 5 most recent items of a List on a page, this is something that you could very easily do through a List View Web Part. Note the __View__ in that title. Remember that List View from earlier ? The one that filtered the items for you beforehand ? Well, you can show this view on a page as well. Separating the Word documents from the Excell files, and showing the 5 most recently changed items in each view on the same page. Ofcourse there are more creative (and useful) ways of using this functionality, but you get my drift.

Web Parts can be used for a lot more, to have a text box that can be used to filter a List View Web Part on a specific value on the page, to show events from a Calendar (also a List), to show content of a different page, to show the people currently online,...

# SharePoint for Users - A detailed loook

## Complete Example

### Structure

So how could a user make all this help him in their day to day business ? Well, lets say we have a teacher, working in a school (obviously) on different projects. One with colleagues form the Science Department to organize an Mathematics event. This teacher also teaches different classes with colleagues (as in they both teach the same class) and they need to share and collaborate on teaching material. For one specific class, the teacher wants to have an environment where the students submit their work, update it during the year, work together with their fellow students on different projects.

Well, the supporting SharePoint platform could be set up like this: 

A central website for teachers. In here, all the teachers can create seperate webs for whatever project, or class they're working on. In our specific case, the teacher would create a website for the Mathematics event, invite fellow teachers involved in the project to the site, use the standard Documents Library to store official documents about the event (Letter templates, Rulebook, Math Questions, Math Questions with answers). 

A different website would hold documents the teacher works on with colleagues for the Physics class. School policy might require that Exam Questions are stored on a central web where there are special permissions and access is logged better, but this web could hold the Course Book, any assignments they're giving their students, a List for marking attendency of the students (pulling in the students Name and Last name from a central location), a calendar where they plan their meetings together and a different library for storing notes about those meetings.

Best practice would advise a different environment for the website where the students have access to to submit their work during the year. The teacher might set up different websites in this main web for each group of students and their specific project and assign only access to the relevant students. Here the students have all the same functionality as the teachers do (Create Lists and Libraries, Pages, maybe even other webs).

See how, at no point in time, the IT department was involved ?

## Using SharePoint

So what other benefits does SharePoint provide the users ? 

There are a issues SharePoint solves that do not immediately meet the eye.

### The Problem

#### Source control

If you're working in an office (any kind office) you probably have to work together with other people. Sometimes this means that you update a file, than some other person needs this file and updates it, and so on and so on. Where is this file stored ? Lets say it's stored on a central file system (sometimes, this is not even the case). Well, you can't allways edit a file directly on this central location and it's not always a good idea either. Why ? Well, let's say you're busy adding a new paragraph at the bottom of the document and someone suddenly decides they don't like some part of the document and updates it. This was all happening on the same document on that central file system. 

Problem #1: Whose version do you think will be kept when they both save it? Yup, whoever saves it last will be the lucky girl or guy. So there's a source control issue here, I'll explain how SharePoint deals with this again afterwards.

#### The one version of the truth

So, very quickly the people that need to edit a file will learn to download the file locally and edit it, safely stored on their local file system where noone else can mess with it. They update it, over time, and after a couple of days they're done. After a while, someone will have forgotten to upload their updated version to the central file system again. In the mean time, other people made changes as well. 

Problem #2: Where is _the one version of the truth_? Where is the most recent version, with all the updates combined ? 

#### Accesibility

Problem #3: Over time, the central document repository has grown so complex that people not using it on a day-to-day basis start having difficulty finding the files they need. A central file repository has little in way of providing context to the person browsing the file system but the folder names.

H:\SchoolOfSciences\Division\Science\Physics\Class2\Assignment1.doc

Pretty logical so far. What about this:

You're looking for a file on safety policy in chemistry rooms and you're looking around:

H:\Teaching\ScienceDivision\ChemistryClass\Year2012\Policies\...

But wait! There's also this folder:

H:\Administration\Safety\Policies\Year2012\Chemistry\...

The one makes sense from a teachers point of view, and the other makes sense from the people making the documents (Administration).

#### Collaboration

Problem #4: Sharing files with colleagues. How are you going to do this with a central file repository? Create a new folder somewhere where you can work on the documents together? But wait, there's no source control, so you have to do it locally. Often people send the files by mail, and even start using their mail box as a file management tool. If you're collaborating, you need to find out who has the latest version (more mails), ultimately sending the file back and forth between team members. This easily leads to the issue of there being multiple version of the same file.

### The SharePoint way

#### Source control

The versioning system of SharePoint takes care of this, allowing users to work on a file together and merge changes (as long as they're not editing the same parts). Or a user can check-out a file, be the sole "owner" of it at the time, and check it back in when he's done.

This is a problem that SharePoint solves in different ways.  This file is stored in SharePoint (the central location again) but SharePoint has version control (even some source control). People __can__ make changes to a file simultaneously, as long as they don't edit the same parts, they can merge their changes. If this is not desired, or it's likely to happen that they edit the same part of the file, you can _check-out_ the file, much like you would check-out a book from a library. There is only one file and you have it. Noone else can edit _that_ file until you check it back in again. 

#### The one version of the truth

SharePoint allows to edit the files online. There simply is no need to download the file locally for editing it. If you have Office installed, editing the file will automatically open your Office client and otherwise it will be a web based client. SharePoint will tell you to save it when closing it and so the file will stay in that central location. Remaining _the one version of the truth_.

#### Accessibility 

What makes sense to one person may not always make sense to other people, sometimes a certain file even makes sense to be stored in multiple locations. With a file structure you usually only have one way of getting to the file. With a website you can link from a certain context to all the files relevant for that group of people. While still having one central file, stored in a path very similar to the one above, it's still a website, so it's very easy to make links in the navigational structure to create a web of related information. 

It should never be difficult to find information (if done correctly).

#### Sharing

Since the file is on a website, it's very easy to share with colleagues. You can easily link to the file, no need to email around a copy of a file to everyone. This helps maintain _the one version of the truth_ as well as save on email clutter as they can just find it in the central location, instead of all the people involved having to email around the file.

If best practice is maintained, you can be very certain that this file is _the one version of the truth_.

There's also archiving (Records Management) and publishing where SharePoint offers a solution.

## Finding the information you need

Metadata

# Summary

SharePoint is a Content Management system and _all_ these features are readily available from a vanilla system. 

SharePoint is meant to allow the users to __manage__ their content themselves. They can create Document Libraries to store their documents __themselves__. They can create the lists and add content to them __themselves__. They can create a complete work environment for their team, their department, their ad-hoc project members, completely by __themselves__. No need to ask the IT department to set up a website, everything is just a couple of clicks away. This is the real _feature_ of SharePoint. Users have the power to do it _themselves_.

Whoever sees SharePoint the first time, as a user, will probably be a little overwhelmed. To be honest, the menu's can get a little crowded, but with a little practice, users should be able to find their way around a SharePoint system.

