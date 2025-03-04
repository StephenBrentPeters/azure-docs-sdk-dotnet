---
title: Azure Cognitive Services Text Analytics client library for .NET
keywords: Azure, dotnet, SDK, API, Azure.AI.TextAnalytics, textanalytics
author: ramya-rao-a
ms.author: ramyar
ms.date: 11/02/2021
ms.topic: reference
ms.prod: azure
ms.technology: azure
ms.devlang: dotnet
ms.service: textanalytics
---

# Azure Cognitive Services Text Analytics client library for .NET - Version 5.2.0-beta.2 

Azure Cognitive Services Text Analytics is a cloud service that provides advanced natural language processing over raw text, and includes the following main features: 
* Language Detection
* Sentiment Analysis
* Key Phrase Extraction
* Entity Recognition (Named, Linked, and Personally Identifiable Information (PII) entities)
* Healthcare Recognition
* Extractive Text Summarization
* Custom Entity Recognition
* Custom Single and Multi Category Classification

[Source code][textanalytics_client_src] | [Package (NuGet)][textanalytics_nuget_package] | [API reference documentation][textanalytics_refdocs] | [Product documentation][textanalytics_docs] | [Samples][textanalytics_samples]

## Getting started

### Install the package
Install the Azure Text Analytics client library for .NET with [NuGet][nuget]:

```dotnetcli
dotnet add package Azure.AI.TextAnalytics
```

This table shows the relationship between SDK versions and supported API versions of the service:

|SDK version|Supported API version of service
|-|- |
|5.2.0-beta.2 | 3.0, 3.1, 3.2-preview.2 (default)
|5.1.0  | 3.0, 3.1 (default)
|5.0.0  | 3.0
|1.0.X | 3.0

### Prerequisites
* An [Azure subscription][azure_sub].
* An existing Cognitive Services or Text Analytics resource.

#### Create a Cognitive Services or Text Analytics resource
Text Analytics supports both [multi-service and single-service access][cognitive_resource_portal]. Create a Cognitive Services resource if you plan to access multiple cognitive services under a single endpoint/key. For Text Analytics access only, create a Text Analytics resource.

You can create either resource using: 

**Option 1:** [Azure Portal][cognitive_resource_portal].

**Option 2:** [Azure CLI][cognitive_resource_cli]. 

Below is an example of how you can create a Text Analytics resource using the CLI:

```PowerShell
# Create a new resource group to hold the Text Analytics resource -
# if using an existing resource group, skip this step
az group create --name <your-resource-name> --location <location>
```

```PowerShell
# Create Text Analytics
az cognitiveservices account create \
    --name <your-resource-name> \
    --resource-group <your-resource-group-name> \
    --kind TextAnalytics \
    --sku <sku> \
    --location <location> \
    --yes
```
For more information about creating the resource or how to get the location and sku information see [here][cognitive_resource_cli].

### Authenticate the client
In order to interact with the Text Analytics service, you'll need to create an instance of the [TextAnalyticsClient][textanalytics_client_class] class. You will need an **endpoint**, and either an **API key** or ``TokenCredential`` to instantiate a client object.  For more information regarding authenticating with cognitive services, see [Authenticate requests to Azure Cognitive Services][cognitive_auth].

#### Get API Key
You can get the `endpoint` and `API key` from the Cognitive Services resource or Text Analytics resource information in the [Azure Portal][azure_portal].

Alternatively, use the [Azure CLI][azure_cli] snippet below to get the API key from the Text Analytics resource.

```PowerShell
az cognitiveservices account keys list --resource-group <your-resource-group-name> --name <your-resource-name>
```

#### Create TextAnalyticsClient with API Key Credential
Once you have the value for the API key, create an `AzureKeyCredential`. This will allow you to
update the API key without creating a new client.

With the value of the endpoint and an `AzureKeyCredential`, you can create the [TextAnalyticsClient][textanalytics_client_class]:

```C# Snippet:CreateTextAnalyticsClient
string endpoint = "<endpoint>";
string apiKey = "<apiKey>";
var client = new TextAnalyticsClient(new Uri(endpoint), new AzureKeyCredential(apiKey));
```

#### Create TextAnalyticsClient with Azure Active Directory Credential

Client API key authentication is used in most of the examples in this getting started guide, but you can also authenticate with Azure Active Directory using the [Azure Identity library][azure_identity].  Note that regional endpoints do not support AAD authentication. Create a [custom subdomain][custom_subdomain] for your resource in order to use this type of authentication.  

To use the [DefaultAzureCredential][DefaultAzureCredential] provider shown below,
or other credential providers provided with the Azure SDK, please install the Azure.Identity package:

```PowerShell
Install-Package Azure.Identity
```

You will also need to [register a new AAD application][register_aad_app] and [grant access][aad_grant_access] to Text Analytics by assigning the `"Cognitive Services User"` role to your service principal.

Set the values of the client ID, tenant ID, and client secret of the AAD application as environment variables: AZURE_CLIENT_ID, AZURE_TENANT_ID, AZURE_CLIENT_SECRET.

```C# Snippet:CreateTextAnalyticsClientTokenCredential
string endpoint = "<endpoint>";
var client = new TextAnalyticsClient(new Uri(endpoint), new DefaultAzureCredential());
```

## Key concepts

### TextAnalyticsClient
A `TextAnalyticsClient` is the primary interface for developers using the Text Analytics client library.  It provides both synchronous and asynchronous operations to access a specific use of Text Analytics, such as language detection or key phrase extraction. 

### Input
A **document**, is a single unit of input to be analyzed by the predictive models in the Text Analytics service.  Operations on `TextAnalyticsClient` may take a single document or a collection of documents to be analyzed as a batch.
For document length limits, maximum batch size, and supported text encoding see [here][data_limits].

### Operation on multiple documents
For each supported operation, `TextAnalyticsClient` provides a method that accepts a batch of documents as strings, or a batch of either `TextDocumentInput` or `DetectLanguageInput` objects. This methods allow callers to give each document a unique ID, indicate that the documents in the batch are written in different languages, or provide a country hint about the language of the document.

**Note:** It is recommended to use the batch methods when working on production environments as they allow you to send one request with multiple documents. This is more performant than sending a request per each document.

### Return value
Return values, such as `AnalyzeSentimentResult`, is the result of a Text Analytics operation, containing a prediction or predictions about a single document.  An operation's return value also may optionally include information about the document and how it was processed.

### Return value Collection
A Return value collection, such as `AnalyzeSentimentResultCollection`, is a collection of operation results, where each corresponds to one of the documents provided in the input batch.  A document and its result will have the same index in the input and result collections. The return value also contains a `HasError` property that allows to identify if an operation executed was succesful or unsuccesful for the given document. It may optionally include information about the document batch and how it was processed.

### Long-Running Operations

For large documents which take a long time to execute, these operations are implemented as [**long-running operations**][dotnet_lro_guidelines].  Long-running operations consist of an initial request sent to the service to start an operation, followed by polling the service at intervals to determine whether the operation has completed or failed, and if it has succeeded, to get the result.

For long running operations in the Azure SDK, the client exposes a `Start<operation-name>` method that returns an `Operation<T>` or a `PageableOperation<T>`.  You can use the extension method `WaitForCompletionAsync()` to wait for the operation to complete and obtain its result.  A sample code snippet is provided to illustrate using long-running operations [below](#recognize-healthcare-entities-asynchronously).

### Thread safety
We guarantee that all client instance methods are thread-safe and independent of each other ([guideline](https://azure.github.io/azure-sdk/dotnet_introduction.html#dotnet-service-methods-thread-safety)). This ensures that the recommendation of reusing client instances is always safe, even across threads.

### Additional concepts
<!-- CLIENT COMMON BAR -->
[Client options](https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/core/Azure.Core/README.md#configuring-service-clients-using-clientoptions) |
[Accessing the response](https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/core/Azure.Core/README.md#accessing-http-response-details-using-responset) |
[Handling failures](https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/core/Azure.Core/README.md#reporting-errors-requestfailedexception) |
[Diagnostics](https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/core/Azure.Core/samples/Diagnostics.md) |
[Mocking](https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/core/Azure.Core/README.md#mocking) |
[Client lifetime](https://devblogs.microsoft.com/azure-sdk/lifetime-management-and-thread-safety-guarantees-of-azure-sdk-net-clients/)
<!-- CLIENT COMMON BAR -->

## Examples
The following section provides several code snippets using the `client` [created above](#create-textanalyticsclient-with-azure-active-directory-credential), and covers the main features of Text Analytics. Although most of the snippets below make use of synchronous service calls, keep in mind that the Azure.AI.TextAnalytics package supports both synchronous and asynchronous APIs.

### Sync examples
* [Detect Language](#detect-language)
* [Analyze Sentiment](#analyze-sentiment)
* [Extract Key Phrases](#extract-key-phrases)
* [Recognize Entities](#recognize-entities)
* [Recognize PII Entities](#recognize-pii-entities)
* [Recognize Linked Entities](#recognize-linked-entities)

### Async examples
* [Detect Language Asynchronously](#detect-language-asynchronously)
* [Recognize Entities Asynchronously](#recognize-entities-asynchronously)
* [Analyze Healthcare Entities Asynchronously](#analyze-healthcare-entities-asynchronously)
* [Run multiple actions Asynchronously](#run-multiple-actions-asynchronously)
* [Perform Extractive Text Summarization Asynchronously](#perform-extractive-text-summarization-asynchronously)

### Detect Language
Run a Text Analytics predictive model to determine the language that the passed-in document or batch of documents are written in.

```C# Snippet:DetectLanguage
string document = @"Este documento está escrito en un idioma diferente al Inglés. Tiene como objetivo demostrar
                    cómo invocar el método de Detección de idioma del servicio de Text Analytics en Microsoft Azure.
                    También muestra cómo acceder a la información retornada por el servicio. Esta capacidad es útil
                    para los sistemas de contenido que recopilan texto arbitrario, donde el idioma es desconocido.
                    La característica Detección de idioma puede detectar una amplia gama de idiomas, variantes,
                    dialectos y algunos idiomas regionales o culturales.";

try
{
    Response<DetectedLanguage> response = client.DetectLanguage(document);

    DetectedLanguage language = response.Value;
    Console.WriteLine($"Detected language {language.Name} with confidence score {language.ConfidenceScore}.");
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```
For samples on using the production recommended option `DetectLanguageBatch` see [here][detect_language_sample].

Please refer to the service documentation for a conceptual discussion of [language detection][language_detection].

### Analyze Sentiment
Run a Text Analytics predictive model to identify the positive, negative, neutral or mixed sentiment contained in the passed-in document or batch of documents.

```C# Snippet:AnalyzeSentiment
string document = @"I had the best day of my life. I decided to go sky-diving and it
                    made me appreciate my whole life so much more.
                    I developed a deep-connection with my instructor as well, and I
                    feel as if I've made a life-long friend in her.";

try
{
    Response<DocumentSentiment> response = client.AnalyzeSentiment(document);
    DocumentSentiment docSentiment = response.Value;

    Console.WriteLine($"Sentiment was {docSentiment.Sentiment}, with confidence scores: ");
    Console.WriteLine($"  Positive confidence score: {docSentiment.ConfidenceScores.Positive}.");
    Console.WriteLine($"  Neutral confidence score: {docSentiment.ConfidenceScores.Neutral}.");
    Console.WriteLine($"  Negative confidence score: {docSentiment.ConfidenceScores.Negative}.");
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```
For samples on using the production recommended option `AnalyzeSentimentBatch` see [here][analyze_sentiment_sample].

To get more granular information about the opinions related to targets of a product/service, also known as Aspect-based Sentiment Analysis in Natural Language Processing (NLP), see a sample on sentiment analysis with opinion mining [here][analyze_sentiment_opinion_mining_sample].

Please refer to the service documentation for a conceptual discussion of [sentiment analysis][sentiment_analysis].

### Extract Key Phrases
Run a model to identify a collection of significant phrases found in the passed-in document or batch of documents.

```C# Snippet:ExtractKeyPhrases
string document = @"My cat might need to see a veterinarian. It has been sneezing more than normal, and although my 
                    little sister thinks it is funny, I am worried it has the cold that I got last week.
                    We are going to call tomorrow and try to schedule an appointment for this week. Hopefully it
                    will be covered by the cat's insurance.
                    It might be good to not let it sleep in my room for a while.";

try
{
    Response<KeyPhraseCollection> response = client.ExtractKeyPhrases(document);
    KeyPhraseCollection keyPhrases = response.Value;

    Console.WriteLine($"Extracted {keyPhrases.Count} key phrases:");
    foreach (string keyPhrase in keyPhrases)
    {
        Console.WriteLine($"  {keyPhrase}");
    }
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```
For samples on using the production recommended option `ExtractKeyPhrasesBatch` see [here][extract_key_phrases_sample].

Please refer to the service documentation for a conceptual discussion of [key phrase extraction][key_phrase_extraction].

### Recognize Entities
Run a predictive model to identify a collection of named entities in the passed-in document or batch of documents and categorize those entities into categories such as person, location, or organization.  For more information on available categories, see [Text Analytics Named Entity Categories][named_entities_categories].

```C# Snippet:RecognizeEntities
string document = @"We love this trail and make the trip every year. The views are breathtaking and well
                    worth the hike! Yesterday was foggy though, so we missed the spectacular views.
                    We tried again today and it was amazing. Everyone in my family liked the trail although
                    it was too challenging for the less athletic among us.
                    Not necessarily recommended for small children.
                    A hotel close to the trail offers services for childcare in case you want that.";

try
{
    Response<CategorizedEntityCollection> response = client.RecognizeEntities(document);
    CategorizedEntityCollection entitiesInDocument = response.Value;

    Console.WriteLine($"Recognized {entitiesInDocument.Count} entities:");
    foreach (CategorizedEntity entity in entitiesInDocument)
    {
        Console.WriteLine($"  Text: {entity.Text}");
        Console.WriteLine($"  Offset: {entity.Offset}");
        Console.WriteLine($"  Length: {entity.Length}");
        Console.WriteLine($"  Category: {entity.Category}");
        if (!string.IsNullOrEmpty(entity.SubCategory))
            Console.WriteLine($"  SubCategory: {entity.SubCategory}");
        Console.WriteLine($"  Confidence score: {entity.ConfidenceScore}");
        Console.WriteLine("");
    }
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```
For samples on using the production recommended option `RecognizeEntitiesBatch` see [here][recognize_entities_sample].

Please refer to the service documentation for a conceptual discussion of [named entity recognition][named_entity_recognition].

### Recognize PII Entities
Run a predictive model to identify a collection of entities containing Personally Identifiable Information found in the passed-in document or batch of documents, and categorize those entities into categories such as US social security number, drivers license number, or credit card number.

```C# Snippet:RecognizePiiEntities
string document = @"Parker Doe has repaid all of their loans as of 2020-04-25.
                    Their SSN is 859-98-0987. To contact them, use their phone number 800-102-1100.
                    They are originally from Brazil and have document ID number 998.214.865-68";

try
{
    Response<PiiEntityCollection> response = client.RecognizePiiEntities(document);
    PiiEntityCollection entities = response.Value;

    Console.WriteLine($"Redacted Text: {entities.RedactedText}");
    Console.WriteLine("");
    Console.WriteLine($"Recognized {entities.Count} PII entities:");
    foreach (PiiEntity entity in entities)
    {
        Console.WriteLine($"  Text: {entity.Text}");
        Console.WriteLine($"  Category: {entity.Category}");
        if (!string.IsNullOrEmpty(entity.SubCategory))
            Console.WriteLine($"  SubCategory: {entity.SubCategory}");
        Console.WriteLine($"  Confidence score: {entity.ConfidenceScore}");
        Console.WriteLine("");
    }
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```

For samples on using the production recommended option `RecognizePiiEntitiesBatch` see [here][recognize_pii_entities_sample].

Please refer to the service documentation for supported [PII entity types][pii_entity_type].

### Recognize Linked Entities
Run a predictive model to identify a collection of entities found in the passed-in document or batch of documents, and include information linking the entities to their corresponding entries in a well-known knowledge base.

```C# Snippet:RecognizeLinkedEntities
string document = @"Microsoft was founded by Bill Gates with some friends he met at Harvard. One of his friends,
                    Steve Ballmer, eventually became CEO after Bill Gates as well. Steve Ballmer eventually stepped
                    down as CEO of Microsoft, and was succeeded by Satya Nadella.
                    Microsoft originally moved its headquarters to Bellevue, Washington in Januaray 1979, but is now
                    headquartered in Redmond";

try
{
    Response<LinkedEntityCollection> response = client.RecognizeLinkedEntities(document);
    LinkedEntityCollection linkedEntities = response.Value;

    Console.WriteLine($"Recognized {linkedEntities.Count} entities:");
    foreach (LinkedEntity linkedEntity in linkedEntities)
    {
        Console.WriteLine($"  Name: {linkedEntity.Name}");
        Console.WriteLine($"  Language: {linkedEntity.Language}");
        Console.WriteLine($"  Data Source: {linkedEntity.DataSource}");
        Console.WriteLine($"  URL: {linkedEntity.Url}");
        Console.WriteLine($"  Entity Id in Data Source: {linkedEntity.DataSourceEntityId}");
        foreach (LinkedEntityMatch match in linkedEntity.Matches)
        {
            Console.WriteLine($"    Match Text: {match.Text}");
            Console.WriteLine($"    Offset: {match.Offset}");
            Console.WriteLine($"    Length: {match.Length}");
            Console.WriteLine($"    Confidence score: {match.ConfidenceScore}");
        }
        Console.WriteLine("");
    }
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```
For samples on using the production recommended option `RecognizeLinkedEntitiesBatch` see [here][recognize_linked_entities_sample].

Please refer to the service documentation for a conceptual discussion of [entity linking][named_entity_recognition].

### Detect Language Asynchronously
Run a Text Analytics predictive model to determine the language that the passed-in document or batch of documents are written in.

```C# Snippet:DetectLanguageAsync
string document = @"Este documento está escrito en un idioma diferente al Inglés. Tiene como objetivo demostrar
                    cómo invocar el método de Detección de idioma del servicio de Text Analytics en Microsoft Azure.
                    También muestra cómo acceder a la información retornada por el servicio. Esta capacidad es útil
                    para los sistemas de contenido que recopilan texto arbitrario, donde el idioma es desconocido.
                    La característica Detección de idioma puede detectar una amplia gama de idiomas, variantes,
                    dialectos y algunos idiomas regionales o culturales.";

try
{
    Response<DetectedLanguage> response = await client.DetectLanguageAsync(document);

    DetectedLanguage language = response.Value;
    Console.WriteLine($"Detected language {language.Name} with confidence score {language.ConfidenceScore}.");
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```

### Recognize Entities Asynchronously
Run a predictive model to identify a collection of named entities in the passed-in document or batch of documents and categorize those entities into categories such as person, location, or organization.  For more information on available categories, see [Text Analytics Named Entity Categories][named_entities_categories].

```C# Snippet:RecognizeEntitiesAsync
string document = @"We love this trail and make the trip every year. The views are breathtaking and well
                    worth the hike! Yesterday was foggy though, so we missed the spectacular views.
                    We tried again today and it was amazing. Everyone in my family liked the trail although
                    it was too challenging for the less athletic among us.
                    Not necessarily recommended for small children.
                    A hotel close to the trail offers services for childcare in case you want that.";

try
{
    Response<CategorizedEntityCollection> response = await client.RecognizeEntitiesAsync(document);
    CategorizedEntityCollection entitiesInDocument = response.Value;

    Console.WriteLine($"Recognized {entitiesInDocument.Count} entities:");
    foreach (CategorizedEntity entity in entitiesInDocument)
    {
        Console.WriteLine($"    Text: {entity.Text}");
        Console.WriteLine($"    Offset: {entity.Offset}");
        Console.WriteLine($"  Length: {entity.Length}");
        Console.WriteLine($"    Category: {entity.Category}");
        if (!string.IsNullOrEmpty(entity.SubCategory))
            Console.WriteLine($"    SubCategory: {entity.SubCategory}");
        Console.WriteLine($"    Confidence score: {entity.ConfidenceScore}");
        Console.WriteLine("");
    }
}
catch (RequestFailedException exception)
{
    Console.WriteLine($"Error Code: {exception.ErrorCode}");
    Console.WriteLine($"Message: {exception.Message}");
}
```

### Analyze Healthcare Entities Asynchronously
Text Analytics for health is a containerized service that extracts and labels relevant medical information from unstructured texts such as doctor's notes, discharge summaries, clinical documents, and electronic health records. For more information see [How to: Use Text Analytics for health][healthcare].

```C# Snippet:TextAnalyticsAnalyzeHealthcareEntitiesConvenienceAsyncAll
// get input documents
string document1 = @"RECORD #333582770390100 | MH | 85986313 | | 054351 | 2/14/2001 12:00:00 AM | CORONARY ARTERY DISEASE | Signed | DIS | \
                    Admission Date: 5/22/2001 Report Status: Signed Discharge Date: 4/24/2001 ADMISSION DIAGNOSIS: CORONARY ARTERY DISEASE. \
                    HISTORY OF PRESENT ILLNESS: The patient is a 54-year-old gentleman with a history of progressive angina over the past several months. \
                    The patient had a cardiac catheterization in July of this year revealing total occlusion of the RCA and 50% left main disease ,\
                    with a strong family history of coronary artery disease with a brother dying at the age of 52 from a myocardial infarction and \
                    another brother who is status post coronary artery bypass grafting. The patient had a stress echocardiogram done on July , 2001 , \
                    which showed no wall motion abnormalities , but this was a difficult study due to body habitus. The patient went for six minutes with \
                    minimal ST depressions in the anterior lateral leads , thought due to fatigue and wrist pain , his anginal equivalent. Due to the patient's \
                    increased symptoms and family history and history left main disease with total occasional of his RCA was referred for revascularization with open heart surgery.";

string document2 = "Prescribed 100mg ibuprofen, taken twice daily.";

// prepare analyze operation input
List<string> batchInput = new List<string>()
{
    document1,
    document2
};

// start analysis process
AnalyzeHealthcareEntitiesOperation healthOperation = await client.StartAnalyzeHealthcareEntitiesAsync(batchInput);
await healthOperation.WaitForCompletionAsync();

// view operation status
Console.WriteLine($"Created On   : {healthOperation.CreatedOn}");
Console.WriteLine($"Expires On   : {healthOperation.ExpiresOn}");
Console.WriteLine($"Status       : {healthOperation.Status}");
Console.WriteLine($"Last Modified: {healthOperation.LastModified}");

// view operation results
await foreach (AnalyzeHealthcareEntitiesResultCollection documentsInPage in healthOperation.Value)
{
    Console.WriteLine($"Results of Azure Text Analytics \"Healthcare Async\" Model, version: \"{documentsInPage.ModelVersion}\"");
    Console.WriteLine("");

    foreach (AnalyzeHealthcareEntitiesResult entitiesInDoc in documentsInPage)
    {
        if (!entitiesInDoc.HasError)
        {
            foreach (var entity in entitiesInDoc.Entities)
            {
                // view recognized healthcare entities
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  NormalizedText: {entity.NormalizedText}");
                Console.WriteLine($"  Links:");

                // view entity data sources
                foreach (EntityDataSource entityDataSource in entity.DataSources)
                {
                    Console.WriteLine($"    Entity ID in Data Source: {entityDataSource.EntityId}");
                    Console.WriteLine($"    DataSource: {entityDataSource.Name}");
                }

                // view assertion
                if (entity.Assertion != null)
                {
                    Console.WriteLine($"  Assertions:");

                    if (entity.Assertion?.Association != null)
                    {
                        Console.WriteLine($"    Association: {entity.Assertion?.Association}");
                    }

                    if (entity.Assertion?.Certainty != null)
                    {
                        Console.WriteLine($"    Certainty: {entity.Assertion?.Certainty}");
                    }
                    if (entity.Assertion?.Conditionality != null)
                    {
                        Console.WriteLine($"    Conditionality: {entity.Assertion?.Conditionality}");
                    }
                }
            }

            Console.WriteLine($"  We found {entitiesInDoc.EntityRelations.Count} relations in the current document:");
            Console.WriteLine("");

            // view recognized healthcare relations
            foreach (HealthcareEntityRelation relations in entitiesInDoc.EntityRelations)
            {
                Console.WriteLine($"    Relation: {relations.RelationType}");
                Console.WriteLine($"    For this relation there are {relations.Roles.Count} roles");

                // view relation roles
                foreach (HealthcareEntityRelationRole role in relations.Roles)
                {
                    Console.WriteLine($"      Role Name: {role.Name}");

                    Console.WriteLine($"      Associated Entity Text: {role.Entity.Text}");
                    Console.WriteLine($"      Associated Entity Category: {role.Entity.Category}");
                    Console.WriteLine("");
                }

                Console.WriteLine("");
            }
        }
        else
        {
            Console.WriteLine("  Error!");
            Console.WriteLine($"  Document error code: {entitiesInDoc.Error.ErrorCode}.");
            Console.WriteLine($"  Message: {entitiesInDoc.Error.Message}");
        }

        Console.WriteLine("");
    }
}
```

### Run multiple actions Asynchronously
This functionality allows running multiple actions in one or more documents. Actions include entity recognition, linked entity recognition, key phrase extraction, Personally Identifiable Information (PII) Recognition, sentiment analysis, and Extractive Text Summarization. For more information see [Using analyze][analyze_operation_howto].

```C# Snippet:AnalyzeOperationConvenienceAsync
    string documentA = @"We love this trail and make the trip every year. The views are breathtaking and well
                        worth the hike! Yesterday was foggy though, so we missed the spectacular views.
                        We tried again today and it was amazing. Everyone in my family liked the trail although
                        it was too challenging for the less athletic among us.";

    string documentB = @"Last week we stayed at Hotel Foo to celebrate our anniversary. The staff knew about
                        our anniversary so they helped me organize a little surprise for my partner.
                        The room was clean and with the decoration I requested. It was perfect!";

    var batchDocuments = new List<string>
    {
        documentA,
        documentB
    };

    TextAnalyticsActions actions = new TextAnalyticsActions()
    {
        ExtractKeyPhrasesActions = new List<ExtractKeyPhrasesAction>() { new ExtractKeyPhrasesAction() },
        RecognizeEntitiesActions = new List<RecognizeEntitiesAction>() { new RecognizeEntitiesAction() },
        DisplayName = "AnalyzeOperationSample"
    };

    AnalyzeActionsOperation operation = await client.StartAnalyzeActionsAsync(batchDocuments, actions);

    await operation.WaitForCompletionAsync();

    Console.WriteLine($"Status: {operation.Status}");
    Console.WriteLine($"Created On: {operation.CreatedOn}");
    Console.WriteLine($"Expires On: {operation.ExpiresOn}");
    Console.WriteLine($"Last modified: {operation.LastModified}");
    if (!string.IsNullOrEmpty(operation.DisplayName))
        Console.WriteLine($"Display name: {operation.DisplayName}");
    Console.WriteLine($"Total actions: {operation.ActionsTotal}");
    Console.WriteLine($"  Succeeded actions: {operation.ActionsSucceeded}");
    Console.WriteLine($"  Failed actions: {operation.ActionsFailed}");
    Console.WriteLine($"  In progress actions: {operation.ActionsInProgress}");

    await foreach (AnalyzeActionsResult documentsInPage in operation.Value)
    {
        IReadOnlyCollection<ExtractKeyPhrasesActionResult> keyPhrasesResults = documentsInPage.ExtractKeyPhrasesResults;
        IReadOnlyCollection<RecognizeEntitiesActionResult> entitiesResults = documentsInPage.RecognizeEntitiesResults;

        Console.WriteLine("Recognized Entities");
        int docNumber = 1;
        foreach (RecognizeEntitiesActionResult entitiesActionResults in entitiesResults)
        {
            Console.WriteLine($" Action name: {entitiesActionResults.ActionName}");
            foreach (RecognizeEntitiesResult documentResults in entitiesActionResults.DocumentsResults)
            {
                Console.WriteLine($" Document #{docNumber++}");
                Console.WriteLine($"  Recognized the following {documentResults.Entities.Count} entities:");

                foreach (CategorizedEntity entity in documentResults.Entities)
                {
                    Console.WriteLine($"  Entity: {entity.Text}");
                    Console.WriteLine($"  Category: {entity.Category}");
                    Console.WriteLine($"  Offset: {entity.Offset}");
                    Console.WriteLine($"  Length: {entity.Length}");
                    Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                    Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                }
                Console.WriteLine("");
            }
        }

        Console.WriteLine("Key Phrases");
        docNumber = 1;
        foreach (ExtractKeyPhrasesActionResult keyPhrasesActionResult in keyPhrasesResults)
        {
            foreach (ExtractKeyPhrasesResult documentResults in keyPhrasesActionResult.DocumentsResults)
            {
                Console.WriteLine($" Document #{docNumber++}");
                Console.WriteLine($"  Recognized the following {documentResults.KeyPhrases.Count} Keyphrases:");

                foreach (string keyphrase in documentResults.KeyPhrases)
                {
                    Console.WriteLine($"  {keyphrase}");
                }
                Console.WriteLine("");
            }
        }
    }
}
```

### Perform Extractive Text Summarization Asynchronously
Get a summary for the input documents by extracting their most relevant sentences. Note that this API can only be used as part of an [Analyze Operation](#run-multiple-actions-asynchronously).

```C# Snippet:TextAnalyticsExtractSummaryWithoutErrorHandlingAsync
// Get input document.
string document = @"Windows 365 was in the works before COVID-19 sent companies around the world on a scramble to secure solutions to support employees suddenly forced to work from home, but “what really put the firecracker behind it was the pandemic, it accelerated everything,” McKelvey said. She explained that customers were asking, “’How do we create an experience for people that makes them still feel connected to the company without the physical presence of being there?”
                    In this new world of Windows 365, remote workers flip the lid on their laptop, bootup the family workstation or clip a keyboard onto a tablet, launch a native app or modern web browser and login to their Windows 365 account.From there, their Cloud PC appears with their background, apps, settings and content just as they left it when they last were last there – in the office, at home or a coffee shop.
                    And then, when you’re done, you’re done.You won’t have any issues around security because you’re not saving anything on your device,” McKelvey said, noting that all the data is stored in the cloud.
                    The ability to login to a Cloud PC from anywhere on any device is part of Microsoft’s larger strategy around tailoring products such as Microsoft Teams and Microsoft 365 for the post-pandemic hybrid workforce of the future, she added. It enables employees accustomed to working from home to continue working from home; it enables companies to hire interns from halfway around the world; it allows startups to scale without requiring IT expertise.
                    “I think this will be interesting for those organizations who, for whatever reason, have shied away from virtualization.This is giving them an opportunity to try it in a way that their regular, everyday endpoint admin could manage,” McKelvey said.
                    The simplicity of Windows 365 won over Dean Wells, the corporate chief information officer for the Government of Nunavut. His team previously attempted to deploy a traditional virtual desktop infrastructure and found it inefficient and unsustainable given the limitations of low-bandwidth satellite internet and the constant need for IT staff to manage the network and infrastructure.
                    We didn’t run it for very long,” he said. “It didn’t turn out the way we had hoped.So, we actually had terminated the project and rolled back out to just regular PCs.”
                    He re-evaluated this decision after the Government of Nunavut was hit by a ransomware attack in November 2019 that took down everything from the phone system to the government’s servers. Microsoft helped rebuild the system, moving the government to Teams, SharePoint, OneDrive and Microsoft 365. Manchester’s team recruited the Government of Nunavut to pilot Windows 365. Wells was intrigued, especially by the ability to manage the elastic workforce securely and seamlessly.
                    “The impact that I believe we are finding, and the impact that we’re going to find going forward, is being able to access specialists from outside the territory and organizations outside the territory to come in and help us with our projects, being able to get people on staff with us to help us deliver the day-to-day expertise that we need to run the government,” he said.
                    “Being able to improve healthcare, being able to improve education, economic development is going to improve the quality of life in the communities.”";

// Prepare analyze operation input. You can add multiple documents to this list and perform the same
// operation to all of them.
var batchInput = new List<string>
{
    document
};

TextAnalyticsActions actions = new TextAnalyticsActions()
{
    ExtractSummaryActions = new List<ExtractSummaryAction>() { new ExtractSummaryAction() }
};

// Start analysis process.
AnalyzeActionsOperation operation = await client.StartAnalyzeActionsAsync(batchInput, actions);

await operation.WaitForCompletionAsync();

// View operation status.
Console.WriteLine($"AnalyzeActions operation has completed");
Console.WriteLine();

Console.WriteLine($"Created On   : {operation.CreatedOn}");
Console.WriteLine($"Expires On   : {operation.ExpiresOn}");
Console.WriteLine($"Id           : {operation.Id}");
Console.WriteLine($"Status       : {operation.Status}");
Console.WriteLine($"Last Modified: {operation.LastModified}");
Console.WriteLine();

// View operation results.
await foreach (AnalyzeActionsResult documentsInPage in operation.Value)
{
    IReadOnlyCollection<ExtractSummaryActionResult> summaryResults = documentsInPage.ExtractSummaryResults;

    foreach (ExtractSummaryActionResult summaryActionResults in summaryResults)
    {
        foreach (ExtractSummaryResult documentResults in summaryActionResults.DocumentsResults)
        {
            Console.WriteLine($"  Extracted the following {documentResults.Sentences.Count} sentence(s):");
            Console.WriteLine();

            foreach (SummarySentence sentence in documentResults.Sentences)
            {
                Console.WriteLine($"  Sentence: {sentence.Text}");
                Console.WriteLine($"  Rank Score: {sentence.RankScore}");
                Console.WriteLine($"  Offset: {sentence.Offset}");
                Console.WriteLine($"  Length: {sentence.Length}");
                Console.WriteLine();
            }
        }
    }
}
```

## Troubleshooting

### General
When you interact with the Cognitive Services Text Analytics client library using the .NET SDK, errors returned by the service correspond to the same HTTP status codes returned for [REST API][textanalytics_rest_api] requests.

For example, if you submit a batch of text document inputs containing duplicate document ids, a `400` error is returned, indicating "Bad Request".

```C# Snippet:BadRequest
try
{
    DetectedLanguage result = client.DetectLanguage(document);
}
catch (RequestFailedException e)
{
    Console.WriteLine(e.ToString());
}
```

You will notice that additional information is logged, like the client request ID of the operation.

```
Message:
    Azure.RequestFailedException:
    Status: 400 (Bad Request)

Content:
    {"error":{"code":"InvalidRequest","innerError":{"code":"InvalidDocument","message":"Request contains duplicated Ids. Make sure each document has a unique Id."},"message":"Invalid document in request."}}

Headers:
    Transfer-Encoding: chunked
    x-aml-ta-request-id: 146ca04a-af54-43d4-9872-01a004bee5f8
    X-Content-Type-Options: nosniff
    x-envoy-upstream-service-time: 6
    apim-request-id: c650acda-2b59-4ff7-b96a-e316442ea01b
    Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
    Date: Wed, 18 Dec 2019 16:24:52 GMT
    Content-Type: application/json; charset=utf-8
```

### Setting up console logging
The simplest way to see the logs is to enable the console logging.
To create an Azure SDK log listener that outputs messages to console use AzureEventSourceListener.CreateConsoleLogger method.

```
// Setup a listener to monitor logged events.
using AzureEventSourceListener listener = AzureEventSourceListener.CreateConsoleLogger();
```

To learn more about other logging mechanisms see [here][logging].

## Next steps

Samples showing how to use the Cognitive Services Text Analytics library are available in this GitHub repository.
Samples are provided for each main functional area, and for each area, samples are provided for analyzing a single document, and a collection of documents in both sync and async mode.

- [Detect Language][detect_language_sample]
- [Analyze Sentiment][analyze_sentiment_sample]
- [Extract Key Phrases][extract_key_phrases_sample]
- [Recognize Entities][recognize_entities_sample]
- [Recognize PII Entities][recognize_pii_entities_sample]
- [Recognize Linked Entities][recognize_linked_entities_sample]
- [Recognize Healthcare Entities][analyze_healthcare_sample]
- [Perform Extractive Text Summarization][extract_summary_sample]
- [Custom Entity Recognition][recognize_custom_entities_sample]
- [Custom Single Category Classification][single_category_classify_sample]
- [Custom Multi Category Classification][multi_category_classify_sample]

### Advanced samples
- [Analyze Sentiment with Opinion Mining][analyze_sentiment_opinion_mining_sample]
- [Run multiple actions][analyze_operation_sample]
- [Create a mock client][mock_client_sample] for testing using the [Moq][moq] library.

## Contributing
See the [CONTRIBUTING.md][contributing] for details on building, testing, and contributing to this library.

This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit [cla.microsoft.com][cla].

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct][code_of_conduct]. For more information see the [Code of Conduct FAQ][coc_faq] or contact [opencode@microsoft.com][coc_contact] with any additional questions or comments.

![Impressions](https://azure-sdk-impressions.azurewebsites.net/api/impressions/azure-sdk-for-net%2Fsdk%2Ftextanalytics%2FAzure.AI.TextAnalytics%2FREADME.png)


<!-- LINKS -->
[textanalytics_client_src]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/src
[textanalytics_docs]: https://docs.microsoft.com/azure/cognitive-services/Text-Analytics/
[textanalytics_refdocs]: https://aka.ms/azsdk-net-textanalytics-ref-docs
[textanalytics_nuget_package]: https://www.nuget.org/packages/Azure.AI.TextAnalytics
[textanalytics_samples]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/README.md
[textanalytics_rest_api]: https://aka.ms/azsdk/textanalytics/restapi
[cognitive_resource_portal]: https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account
[cognitive_resource_cli]: https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account-cli
[dotnet_lro_guidelines]: https://azure.github.io/azure-sdk/dotnet_introduction.html#dotnet-longrunning

[analyze_healthcare_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample7_AnalyzeHealthcareEntities.md
[analyze_operation_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample_AnalyzeActions.md
[analyze_operation_howto]: https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-call-api?tabs=synchronous#using-the-api-asynchronously
[healthcare]: https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-for-health?tabs=ner
[language_detection]: https://docs.microsoft.com/azure/cognitive-services/Text-Analytics/how-tos/text-analytics-how-to-language-detection
[sentiment_analysis]: https://docs.microsoft.com/azure/cognitive-services/Text-Analytics/how-tos/text-analytics-how-to-sentiment-analysis
[key_phrase_extraction]: https://docs.microsoft.com/azure/cognitive-services/Text-Analytics/how-tos/text-analytics-how-to-keyword-extraction
[named_entity_recognition]: https://docs.microsoft.com/azure/cognitive-services/Text-Analytics/how-tos/text-analytics-how-to-entity-linking
[named_entities_categories]: https://docs.microsoft.com/azure/cognitive-services/Text-Analytics/named-entity-types
[pii_entity_type]:https://docs.microsoft.com/azure/cognitive-services/text-analytics/named-entity-types?tabs=personal 

[textanalytics_client_class]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/src/TextAnalyticsClient.cs
[azure_identity]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/identity/Azure.Identity
[cognitive_auth]: https://docs.microsoft.com/azure/cognitive-services/authentication
[register_aad_app]: https://docs.microsoft.com/azure/cognitive-services/authentication#assign-a-role-to-a-service-principal
[aad_grant_access]: https://docs.microsoft.com/azure/cognitive-services/authentication#assign-a-role-to-a-service-principal
[custom_subdomain]: https://docs.microsoft.com/azure/cognitive-services/authentication#create-a-resource-with-a-custom-subdomain
[DefaultAzureCredential]: https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/identity/Azure.Identity/README.md#defaultazurecredential
[logging]: https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/core/Azure.Core/samples/Diagnostics.md
[data_limits]: https://docs.microsoft.com/azure/cognitive-services/text-analytics/concepts/data-limits?tabs=version-3
[contributing]: https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.TextAnalytics_5.2.0-beta.2/CONTRIBUTING.md

[detect_language_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample1_DetectLanguage.md
[analyze_sentiment_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample2_AnalyzeSentiment.md
[analyze_sentiment_opinion_mining_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample2.1_AnalyzeSentimentWithOpinionMining.md
[extract_key_phrases_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample3_ExtractKeyPhrases.md
[extract_summary_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample8_ExtractSummary.md
[recognize_entities_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample4_RecognizeEntities.md
[recognize_pii_entities_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample5_RecognizePiiEntities.md
[recognize_linked_entities_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample6_RecognizeLinkedEntities.md
[mock_client_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample_MockClient.md
[recognize_custom_entities_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample9_RecognizeCustomEntities.md
[single_category_classify_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample10_SingleCategoryClassify.md
[multi_category_classify_sample]: https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.2.0-beta.2/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample11_MultiCategoryClassify.md

[azure_cli]: https://docs.microsoft.com/cli/azure
[azure_sub]: https://azure.microsoft.com/free/dotnet/
[nuget]: https://www.nuget.org/
[azure_portal]: https://portal.azure.com
[moq]: https://github.com/Moq/moq4/

[cla]: https://cla.microsoft.com
[code_of_conduct]: https://opensource.microsoft.com/codeofconduct/
[coc_faq]: https://opensource.microsoft.com/codeofconduct/faq/
[coc_contact]: mailto:opencode@microsoft.com

