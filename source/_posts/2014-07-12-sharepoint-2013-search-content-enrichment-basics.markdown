---
layout: post
title: "SharePoint 2013 - Search - Content Enrichment - Basics"
date: 2014-07-12 15:07
comments: true
categories: 
- sharepoint
- search
---

## Content Enrichment Service - Output Properties

We've implemented a SP 2013 Content Enrichment service at a client in the last week and I'd like to share some things you need to watch out for when creating your own.

Especially since there's a lot of documentation out there, and a lot of documentation that's missing. More specifically how the Search Engine deals with edge cases when calling and processing the result of your service.

### Optional or required ?

For instance: OutputProperties of your service, do you **have to** return them ? Are they merely a guideline and can you still return others (wouldn't be logical, but still)? What about returning managed properties that already exist on the record?

A colleague of mine experimented a little and noticed the following things:

  - The specified output properties are *optional*
    - You are not required to return all of the properties listed.
  - The specified output properties are *limiting*
    - If the managed property is not listed, you are not allowed to return it
      - I believe an error will be logged in the ULS

### Returning a managed property that already has a value on the item ?

```
Microsoft.Ceres.Evaluation.DataModel.Types.SchemaException: Cannot add field MyManagedPropertyName to bucket, it already exists.
```

That would mean a solid "no". Although the disassembly has an if case that might allow you to override it, but I can't make much sense of the code to say when it would work. Seems to depend on TypeConversions.IsCompatible.

### Output property type

The managed properties themselves, I had to dig into the DLL's to figure that out. The AbstractProperty class has a static method call that lists the supported property types (as the actual Property types are generic Property<T> types). These are the supported properties :

```
  Property<string>
  Property<int>
  Property<long>
  Property<bool>
  Property<double>
  Property<Decimal>
  Property<DateTime>
  Property<Guid>
  Property<byte[]>
  And their List<T> versions (Property<List<string>>,...)
```
Using any other type in your Content Enrichment service will compile, but will throw errors on the Search Engine side.

You cannot use just any type for a specific Managed Property. If your Managed Property is registered as type bool, your enriched Property<T> will also have to be of type bool (Property<bool>). Again, an appropriate error will be logged in ULS saying that Managed Property expected to be of type T while the type returned was type Z.

### Debugging

This is actually related to anything to do with your custom Content Enrichment Service.

If you want to find CEWS related errors in the ULS logs, they are logged as medium/high (so far as I could tell) and you can see them by filtering on

  - Message contains "ContentEnrichmentClient"
  - The errors are thrown by ContentProcessingEnrichmentClientEvaluator
  - The errors are of type Microsoft.Ceres.Evaluation.DataModel.EvaluationException

Your service is called by an instance of Microsoft.Ceres.ContentProcessing.Evaluators.ContentEnrichmentClientProducer.

None of the errors will be logged as descriptively in the Crawl Log. They will merely say "Failed to process the results returned by the content processing enrichment service" or some such.

### Which items were enriched ?

There's no easy way to just get all the records that were touched by your Content Enrichment Service as far as I know. We've added a managed property in the sense of "IsEnrichedByMyService" of type bool and update that. This way you can also find the amound of successfully enriched items, as they don't get a seperate tab in your Crawl Log like the errors do.



