# Segment 3: Language Service
[link](link)
## Extracting insights from textusing the Language Service

Every day, the world generates a vast quantity of data; much of it text-based in the form of emails, social media posts, online reviews, business documents, and more. Artificial intelligence techniques that apply statistical and semantic models enable you to create applications that extract meaning and insights from this text-based data.

The Language Azure Cognitive Service provides an API for common text analysis techniques that you can easily integrate into your own application code.

In this module, you will learn how to use the Language service to:

* Detect language
* Extract key phrases
* Analyze sentiment
* Extract entities
* Extract linked entities

## Provision a Language resource

The Language service is designed to help you extract information from text. It provides functionality that you can use for:

Language detection - determining the language in which text is written.
Key phrase extraction - identifying important words and phrases in the text that indicate the main points.
Sentiment analysis - quantifying how positive or negative the text is.
Named entity recognition - detecting references to entities, including people, locations, time periods, organizations, and more.
Entity linking - identifying specific entities by providing reference links to Wikipedia articles.

![text-analyitics-resource](https://learn.microsoft.com/en-us/training/wwl-data-ai/extract-insights-text-with-text-analytics-service/media/text-analytics-resource.png)

Azure resources for text analysis
To use the Language service to analyze text, you must provision a resource for it in your Azure subscription. You can provision a single-service Language resource, or you can use a multi-service Cognitive Services resource.

After you have provisioned a suitable resource in your Azure subscription, you can use its endpoint and one of its subscription keys to call the Language APIs from your code. You can call the Language APIs by submitting requests in JSON format to the REST interface, or by using any of the available programming language-specific SDKs.

## Detect a language

The Language Detection API evaluates text input and, for each document submitted, returns language identifiers with a score indicating the strength of the analysis. The Language service recognizes up to 120 languages.

This capability is useful for content stores that collect arbitrary text, where language is unknown. Another scenario could involve a chat bot. If a user starts a session with the chat bot, language detection can be used to determine which language they are using and allow you to configure your bot responses in the appropriate language.

You can parse the results of this analysis to determine which language is used in the input document. The response also returns a score, which reflects the confidence of the model (a value between 0 and 1).

Language detection can work with documents or single phrases. It's important to note that the document size must be under 5,120 characters. The size limit is per document and each collection is restricted to 1,000 items (IDs). A sample of a properly formatted JSON payload that you might submit to the service in the request body is shown here, including a collection of documents, each containing a unique id and the text to be analyzed. Optionally, you can provide a countryHint to improve prediction performance.

```
{
  "documents": [
    {
      "countryHint": "US",
      "id": "1",
      "text": "Hello world"
    },
    {
      "id": "2",
      "text": "Bonjour tout le monde"
    }
  ]
}
```

The service will return a JSON response that contains a result for each document in the request body, including the predicted language and a value indicating the confidence level of the prediction. The confidence level is a value ranging from 0 to 1 with values closer to 1 being a higher confidence level. Here's an example of a standard JSON response that maps to the above request JSON.

```
{
  "documents": [
   {
     "id": "1",
     "detectedLanguage": {
       "name": "English",
       "iso6391Name": "en",
       "confidenceScore": 1
     },
     "warnings": []
   },
   {
     "id": "2",
     "detectedLanguage": {
       "name": "French",
       "iso6391Name": "fr",
       "confidenceScore": 1
     },
     "warnings": []
   }
  ],
  "errors": [],
  "modelVersion": "2020-04-01"
}
```

In our sample, all of the languages show a confidence of 1, mostly because the text is relatively simple and easy to identify the language for.

If you pass in a document that has multilingual content, the service will behave a bit differently. Mixed language content within the same document returns the language with the largest representation in the content, but with a lower positive rating, reflecting the marginal strength of that assessment. In the following example, the input is a blend of English, Spanish, and French. The analyzer uses statistical analysis of the text to determine the predominant language.

```
{
  "documents": [
    {
      "id": "1",
      "text": "Hello, I would like to take a class at your University. ¿Se ofrecen clases en español? Es mi primera lengua y más fácil para escribir. Que diriez-vous des cours en français?"
    }
  ]
}

The following sample shows a response for this multi-language example.

{
  "documents": [
    {
      "id": "1",
      "detectedLanguages": [
        {
          "name": "Spanish",
          "iso6391Name": "es",
          "score": 0.9375
        }
      ]
    }
  ],
  "errors": []
}
```

The last condition to consider is when there is ambiguity as to the language content. The scenario might happen if you submit textual content that the analyzer is not able to parse, for example because of character encoding issues when converting the text to a string variable. As a result, the response for the language name and ISO code will indicate (unknown) and the score value will be returned as NaN, or Not a Number. The following example shows how the response would look.

```
{
      "id": "5",
      "detectedLanguages": [
        {
          "name": "(Unknown)",
          "iso6391Name": "(Unknown)",
          "score": "NaN"
        }
      ]
```

## Extract key phrases

Key phrase extraction is the process of evaluating the text of a document, or documents, and then identifying the main points around the context of the document(s).

Key phrase extraction works best for larger documents (the maximum size that can be analyzed is 5,120 characters).

As with language detection, the REST interface enables you to submit one or more documents for analysis.

```
{
  "documents": [
    {
      "id": "1",
      "language": "en",
      "text": "You must be the change you wish 
               to see in the world."
    },
    {
      "id": "2",
      "language": "en",
      "text": "The journey of a thousand miles 
               begins with a single step."
    }
  ]
}
```

The response contains a list of key phrases detected in each document.

```
{
  "documents": [
   {
     "id": "1",
     "keyPhrases": [
       "change",
       "world"
     ],
     "warnings": []
   },
   {
     "id": "2",
     "keyPhrases": [
       "miles",
       "single step",
       "journey"
     ],
     "warnings": []
   }
  ],
  "errors": [],
  "modelVersion": "2020-04-01"
}
```

## Analyze sentiment

Sentiment analysis is used to evaluate how positive or negative a text document is, which can be useful in various workloads, such as:

Evaluating a movie, book, or product by quantifying sentiment based on reviews.
Prioritizing customer service responses to correspondence received through email or social media messaging.
When using the Language service to evaluate sentiment, the response includes overall document sentiment and individual sentence sentiment for each document submitted to the service.

For example, you could submit a single document for sentiment analysis like this:

```
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Smile! Life is good!"
    }
  ]
}
```

The response from the service might look like this:

```
{
  "documents": [
   {
     "id": "1",
     "sentiment": "positive",
     "confidenceScores": {
       "positive": 0.99,
       "neutral": 0.01,
       "negative": 0.00
     },
     "sentences": [
       {
         "text": "Smile!",
         "sentiment": "positive",
         "confidenceScores": {   
             "positive": 0.97,
	         "neutral": 0.02, 
             "negative": 0.01
           },
         "offset": 0,
         "length": 6
       },
       {
	      "text": "Life is good!",
          "sentiment": "positive",
          "confidenceScores": {   
             "positive": 0.98,
	         "neutral": 0.02,  
             "negative": 0.00
           },
         "offset": 7,
         "length": 13
       }
     ],
     "warnings": []
   }
  ],
  "errors": [],
  "modelVersion": "2020-04-01"
}
```

Sentence sentiment is based on confidence scores for positive, negative, and neutral classification values between 0 and 1.

Overall document sentiment is based on sentences:

If all sentences are neutral, the overall sentiment is neutral.
If sentence classifications include only positive and neutral, the overall sentiment is positive.
If the sentence classifications include only negative and neutral, the overall sentiment is negative.
If the sentence classifications include positive and negative, the overall sentiment is mixed.

## Extract entities

Named Entity Recognition identifies entities that are mentioned in the text. Entities are grouped into categories and subcategories, for example:

* Person
* Location
* DateTime
* Organization
* Address
* Email
* URL

Input for entity recognition is similar to input for other Language API functions:

```
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Joe went to London on Saturday"
    }
  ]
}
```

The response includes a list of categorized entities found in each document:

```
{
  "documents":[
      {
          "id":"1",
          "entities":[
          {
            "text":"Joe",
            "category":"Person",
            "offset":0,
            "length":3,
            "confidenceScore":0.62
          },
          {
            "text":"London",
            "category":"Location",
            "subcategory":"GPE",
            "offset":12,
            "length":6,
            "confidenceScore":0.88
          },
          {
            "text":"Saturday",
            "category":"DateTime",
            "subcategory":"Date",
            "offset":22,
            "length":8,
            "confidenceScore":0.8
          }
        ],
        "warnings":[]
      }
  ],
  "errors":[],
  "modelVersion":"2021-01-15"
}
```

## Extract linked entities

In some cases, the same name might be applicable to more than one entity. For example, does an instance of the word "Venus" refer to the planet or the goddess from mythology?

Entity linking can be used to disambiguate entities of the same name by referencing an article in a knowledge base. Wikipedia provides the knowledge base for the Text Analytics service. Specific article links are determined based on entity context within the text.

For example, "I saw Venus shining in the sky" is associated with the link https://en.wikipedia.org/wiki/Venus; while "Venus, the goddess of beauty" is associated with https://en.wikipedia.org/wiki/Venus_(mythology).

As with all Language service functions, you can submit one or more documents for analysis:
```
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "I saw Venus shining in the sky"
    }
  ]
}
```

The response includes the entities identified in the text along with links to associated articles:

```
{
  "documents":
    [
      {
        "id":"1",
        "entities":[
          {
            "name":"Venus",
            "matches":[
              {
                "text":"Venus",
                "offset":6,
                "length":5,
                "confidenceScore":0.01
              }
            ],
            "language":"en",
            "id":"Venus",
            "url":"https://en.wikipedia.org/wiki/Venus",
            "dataSource":"Wikipedia"
          }
        ],
        "warnings":[]
      }
    ],
  "errors":[],
  "modelVersion":"2020-02-01"
}
```

