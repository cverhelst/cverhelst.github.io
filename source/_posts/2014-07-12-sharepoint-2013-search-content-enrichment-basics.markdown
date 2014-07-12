---
layout: post
title: "SharePoint 2013 - Search - Content Enrichment - Basics"
date: 2014-07-12 15:07
comments: true
categories: 
- sharepoint
- search
---

We've implemented a SP 2013 Content Enrichment service at a client in the last week and I'd like to share some things you need to watch out for when creating your own.

Especially since there's a lot of documentation out there, and a lot of documentation that's missing. More specifically how the Search Engine deals with edge cases when calling and processing the result of your service.

For instance: OutputProperties of your service, do you **have to** return them ? Are they merely a guideline and can you still return others ? What about returning managed properties that already exist ?

A colleague of mine experimented a little and noticed the following things:

  - The specified output properties are *optional*
    - You are not required to return all of the properties listed.
  - The specified output properties are *limiting*
    - If the managed property is not listed, you are not allowed to return it
      - I believe an error will be logged in the ULS

The managed properties themselves, I had to dig into the DLL's to figure that out. The AbstractProperty class has a static method call that lists the supported property types (as the actual Property types are generic Property<T> types). These are the supported properties :

```csharp
  - Property<string>
  - Property<int>
  - Property<long>
  - Property<bool>
  - Property<double>
  - Property<Decimal>
  - Property<DateTime>
  - Property<Guid>
  - Property<byte[]>
  - And their List<T> versions (Property<List<string>>,...)
```
Using any other type in your Content Enrichment service will compile, but will throw errors on the Search Engine side.

You cannot use just any type for a specific Managed Property. If your Managed Property is registered as type bool, your enriched Property<T> will also have to be of type bool (Property<bool>). Again, an appropriate error will be logged in ULS saying that Managed Property expected to be of type T while the type returned was type Z.

If you want to get find these errors in the ULS logs, they are logged as medium/high (so far as I could tell) and you can see them by filtering on

  - Message contains "ContentEnrichmentClient"
  - The errors are thrown by ContentProcessingEnrichmentClientEvaluator
  - The errors are of type Microsoft.Ceres.Evaluation.DataModel.EvaluationException

Your service is called by an instance of Microsoft.Ceres.ContentProcessing.Evaluators.ContentEnrichmentClientProducer.

None of the errors will be logged as descriptively in the Crawl Log. They will merely say "Failed to process the results returned by the content processing enrichment service" or some such.

So another thing we would've liked to know is how to make the the errorcode's we can return in the web service have an associated error message (it's not recommended to actually *throw* errors, just log them and set an appropriate errorcode on the result).

By the grace of SharePoint Search gods there might actually be a way! While reflecting on the DLL's of the Microsoft.Ceres.ContentProcessing.Evaluators.ContentEnrichmentClientProducer (Cameron approves of descriptive class names) I encountered a **public** static method on what seems to be a singleton instance of a sort of ErrorService:

- Microsoft.Ceres.ContentEngine.Util.Exceptions.CtsProcessingErrors
- It's full of constants pointing to the built-in error messages
  - OperatorTimeOut
  - DisplayAuthorFieldNotFound (you see this one in the logs a lot)
  - ContentProcessingEnrichmentInvalidType (Property<Z> for Managed Property T)
  - ContentProcessingEnrichmentFailedToProcessResult

And if you check the flow of the code around the call to your Content Enrichment Service, he used the returned error code if one is provided. And he will log an UnknownErrorIdException if it was not registered on the singleton.

So what I'm wondering now is if the Content Enrichment Service runs in the same app pool as the Search Engine (or at least the client calling our service) does. That would hopefully mean that we can register our own errror codes with this singleton and have meaningful messages in the Crawl Log.

This would probably be done on another Class:

  - Microsoft.Ceres.Evaluation.Services.ErrorDefinitions.ProcessingErrors

which has a concurrent dictionary of errorId mappings, which is called by the CtsProcessingErrors class in it's constructor.

But that will be for another post



