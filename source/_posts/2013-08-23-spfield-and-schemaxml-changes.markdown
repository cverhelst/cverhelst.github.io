---
layout: post
title: "SPField and .SchemaXML changes"
date: 2013-08-23 21:28
comments: true
categories: 
- sharepoint
- fixes
- schemaXml
---

# Backstory

So, a long time ago we ran into an issue where, if we updated the site content type and pushed the changes to the list, we would lose our field title translations. It would just display whatever was the english translation, but for every language.

Likewise, when updating a site column, and pushing down, we got our translations (resources) back, but lost any additional field modifications (required, showindisplayform,...).

Not fun. At all. 

We investigated the issue but couldn't come up with anything better (deadlines, deadlines, deadlines!) than making our changes to the site content type (without pushing down) and then going over each content type usage and making the change there as well.
This still broke the resources but we kept our modifications.

Fixing the resources involved updating every field instance (on the list) that was part of the content type we were updating. Also very much NOT fun. In our case, we had a lot of subwebs all setup the same which would result in 23 webs all needing to have their list fields updated. This takes a while, apparently.

# Cause

Fast forward to now and we've finally figured out what the root cause of this issue is (and how to permenantly fix it).

The reason our translation failed is because during the life-cycle of the SharePoint application deployed at the customer, the SPField's had their SchemaXML's updated manually.
As in, reading out the SchemaXmlWithResourceTokens property, updating the DisplayName, Description and Group properties with a resource token (which it didn't originally have), and updating the field (which is actually unnecessary).

I don't mean to point fingers but this is legacy code and we've always wondered if it wasn't fishy. We now know.

Running some tests of our own, we managed to reproduce the above symptoms by making changes to the SchemaXml of fields.

This brings you in a situation where:

* After the SPField SchemaXml has been updated, none of the parent SPField (the site column) changes are pushed to their children (list fields) any longer.
* Updating the parent Content Type and pushing the changes down breaks these translations (giving you the english translations, all the time). This however, does fix the parent SPField not pushing down its changes.
* Updating the parent SPField does push down its changes, but removes any list level changes that were made before. 

The last 2 points can be alternated and they'll keep giving the same effect (like a loop, the one breaks the fixes of the other). Thats the situation we were in.

# Solution

To get out of this situation you need to remove all the fieldLinks on the parent Content Type for the affected fields, add them again and push the changes down. All your data is still there, no worries. What you do lose is all the local list level changes (if you made any). Updating your parent SPField with the correct configuration and pushing down puts an end to that.

Your translations are there again and you can make changes to the parent field/content type without breaking anything.

# Conclusion

Stay the **** out of the SPField.SchemaXml if you want to have any hope of retaining your sanity while performing maintenance to the application afterwards.
No. Seriously.

I'll add a SharePoint Visual Studio solution that you can deploy to check this for yourself (it has all the required code in features that you can just activate to see the effect for yourself).

# Investigation

Here's the steps we used to test this behavior:

### Base

* Given:
** A field
*** Provisioned from CAML with resource keys
** A contentType
*** Provisioned from CAML to include the field as a fieldLink
** A list
*** Based on the previous contentType
*The situation is as expected:
** Field titles are translated in the forms of the list.

### Change Set 1
* Update: 
** A new field is added through code without resource keys
** This field is also added as a field link to the content type
* This situation is as expected:
** Field exists in the list
** Field title is not translated across UI display language changes.

### Change Set 2
* Update:
** Add resource keys to the field
*** We add the resource keys by changing the SchemaXmlWithResourceTokens in code.
* The situation is as expected:
** The field title is translated.

### Change Set 3
* Update:
** Make the field in the rootWeb required
*** We update the site column field and update with pushing the changes to the list
* The situation is _NOT_ as expected
** The field is not required in the list

### Change Set 4
* Update:
** Make the fieldLink on the content Type required
* The situation is as expected
** The field in the form is required

### Change Set 5
* Update:
** We call update on the rootWeb contentType and push the changes to the list (regardless if any changes were made).
* The situation is _NOT_ as expected
** The field is no longer required
** The field title is no longer translated

### Change Set 6
* Update:
** We call update on the rootWeb field and push the changes to the list (regardless if any changes were made).
* The situation is _NOT_ as expected
** The field is no longer required
** The field titles are translated again

## Change Set 7
* Update:
** Make the field in the rootWeb required again
* The situation is as expected
** The field in the list is also required
** The field titles are translated again

## Change Set 8
* Update:
** We call update on the rootWeb field and push the changes to the list (regardless if any changes were made).
* The situation is not as expected
** The field is still required
** The field titles have lost their translations again