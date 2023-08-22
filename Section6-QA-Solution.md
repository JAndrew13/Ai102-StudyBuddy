# Section 6: Understand question answering
The Language service includes a question answering capability, which enables you to define a knowledge base of question and answer pairs that can be queried using natural language input. The knowledge base can be published to a REST endpoint and consumed by client applications, commonly bots.

The knowledge base can be created from existing sources, including:

* Web sites containing frequently asked question (FAQ) documentation.
* Files containing structured text, such as brochures or user guides.
* Built-in chit chat question and answer pairs that encapsulate common conversational exchanges.

*Note:*
*The question answering capability of the Language service is a newer version of the QnA Service, which still exists as a standalone service. To learn how to migrate a QnA Maker knowledge base to the Language service, see the migration guide.*

## Compare question answering to language understanding

A question answering knowledge base is a form of language model, which raises the question of when to use question answering, and when to use the conversational language understanding capabilities of the Language service.

The two features are similar in that they both enable you to define a language model that can be queried using natural language expressions. However, there are some differences in the use cases that they are designed to address, as shown in the following table:

|                  | Question answering                                                                                   | Language understanding                                                                                               |   |   |
|------------------|------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|---|---|
| Usage pattern    | User submits a question, expecting an answer                                                         | User submits an utterance, expecting an appropriate response or action                                               |   |   |
| Query processing | Service uses natural language understanding to match the question to an answer in the knowledge base | Service uses natural language understanding to interpret the utterance, match it to an intent, and identify entities |   |   |
| Response         | Response is a static answer to a known question                                                      | Response indicates the most likely intent and referenced entities                                                    |   |   |
| Client logic     | Client application typically presents the answer to the user                                         | Client application is responsible for performing appropriate action based on the detected intent                     |   |   |

The two services are in fact complementary. You can build comprehensive natural language solutions that combine conversational language understanding models and question answering knowledge bases.

## Create a knowledge base

To create a question answering solution, you can use the REST API or SDK to write code that defines, trains, and publishes the knowledge base. However, it is more common to use the [Language Studio](https://language.cognitive.azure.com/) web interface to define and manage a knowledge base.

To create a knowledge base:
1. Create a Language resource in your Azure subscription.
    * Enable the question answering feature.
    * Create or select an Azure Cognitive Search resource to host the knowledge base index.

2. In Language Studio, select the Language resource and create a Custom question answering project.

3. Name the knowledge base.

4. Add one or more data sources to populate the knowledge base:
    * URLs for web pages containing FAQs.
    * Files containing structured text from which questions and answers can be derived.
    * Pre-defined chit-chat datasets that include common conversational questions and responses in a specified style.
      
5. Create the knowledge base and edit question and answer pairs in the portal.

## Implement multi-turn conversation
Although you can often create an effective knowledge base that consists of individual question and answer pairs, sometimes you might need to ask follow-up questions to elicit more information from a user before presenting a definitive answer. This kind of interaction is referred to as a multi-turn conversation.

[img](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-qna-solution-qna-maker/media/multi-turn-conversation.png)

You can enable multi-turn responses when importing questions and answers from an existing web page or document based on its structure, or you can explicitly define follow-up prompts and responses for existing question and answer pairs.

For example, suppose an initial question for a travel booking knowledge base is "How can I cancel a reservation?". A reservation might refer to a hotel or a flight, so a follow-up prompt is required to clarify this detail. The answer might consist of text such as "Cancellation policies depend on the type of reservation" and include follow-up prompts with links to answers about canceling flights and canceling hotels.

When you define a follow-up prompt for multi-turn conversation, you can link to an existing answer in the knowledge base or define a new answer specifically for the follow-up. You can also restrict the linked answer so that it is only ever displayed in the context of the multi-turn conversation initiated by the original question.

## Test and publish a knowledge base

After you have defined a knowledge base, you can train its natural language model, and test it before publishing it for use in an application or bot.

### Testing a knowledge base

You can test your knowledge base interactively in Language Studio, submitting questions and reviewing the answers that are returned. You can inspect the results to view their confidence scores as well as other potential answers.

[img](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-qna-solution-qna-maker/media/test-qna.png)

### Deploying a knowledge base
When you are happy with the performance of your knowledge base, you can deploy it to a REST endpoint that client applications can use to submit questions and receive answers.

## Use a knowledge base

To consume the published knowledge base, you can use the REST interface.

The minimal request body for the function contains a question, like this:

```
{
  "question": "What do I need to do to cancel a reservation?"
}
```
The response includes the closest question match that was found in the knowledge base, along with the associated answer, the confidence score, and other metadata about the question and answer pair.
```
{
  "answers": [
    {
      "questions": [
        "How can I cancel a reservation?"
      ],
      "answer": "Call us on 555 123 4567 to cancel a reservation.",
      "confidenceScore": 1.0,
      "id": 6,
      "source": "https://margies-travel.com/faq",
      "metadata": {},
      "dialog": {
        "isContextOnly": false,
        "prompts": []
      }
    }
  ]
}
```
## Improve question answering performance

After creating and testing a knowledge base, you can improve its performance with active learning and by defining synonyms.

### Use active learning
**Active learning** can help you make continuous improvements so that it gets better at answering user questions correctly over time.

Active learning helps improve the knowledge base in two ways:

* **Implicit feedback**: As incoming requests are processed, the service identifies user-provided questions that have multiple, similarly scored matches in the knowledge base. These are automatically clustered as alternate phrase suggestions for the possible answers that you can accept or reject in the Suggestions page for your knowledge base in Language Studio.

* **Explicit feedback**. When developing a client application you can control the number of possible question matches returned for the user's input by specifying the top parameter, as shown here:
```
{
    "question": "I want to book a hotel.",
    "top": 3
}
```
The response from the service includes a question object for each possible match, up to the top value specified in the request:
```
{
    "answers":[
        {"questions":[
            "How do I book a hotel?"
        ],
        "answer":"Call 555-123-4567 to book.",
        "score":76.55,
        "id":2,
        ...
        },
        {"questions":[
            "Can I book multiple hotel rooms?"
        ],
        "answer":"Yes, you can reserve up to 3 rooms.",
        "score":76.15,
        "id":6,
        ...
        },
        {"questions":[
            "Is there a booking fee?"
        ],
        "answer":"No, we do not charge a booking fee.",
        "score":75.99,
        "id":11,
        ...
        }
    ]
}
```
You can implement logic in your client app to compare the score property values for the questions, and potentially present the questions to the user so they can positively identify the question closest to what they intended to ask.

With the correct question identified, your app can use the REST API to send feedback containing suggested alternative phrasing based on the user's original input:
```
{
    "records": [
        {
            "userId": "1234",
            "userQuestion": "I want to book a hotel.",
            "qnaId": 2
        }
    ]
}
```
The **qnaId** in the feedback corresponds to the id of the question the user identified as the correct match. The userId parameter is an identifier for the user and can be any value you choose, such as an email address or numeric identifier.

The feedback will be presented in the active learning Suggestions page for your knowledge base in Language Studio for you to accept or reject.

*note: To learn more about active learning, see the Enrich your project with [active learning](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/tutorials/active-learning).*

### Define synonyms
Synonyms are useful when question submitted by users might include multiple different words to mean the same thing. For example, a travel agency customer might refer to a "reservation" or a "booking". By defining these as synonyms, the question answering service can find an appropriate answer regardless of which term an individual customer uses.

To define synonyms, you must use the REST API to submit synonyms in the following JSON format:
```
{
    "synonyms": [
        {
            "alterations": [
                "reservation",
                "booking",,
                ]
        }
    ]
}
```
*note: To learn more about synonyms, see the [Improve quality of response with synonyms](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/tutorials/adding-synonyms).

## Create a question answering bot
While you can use a question answering knowledge store in any sort of application, they are commonly used to support bots.

A bot is a conversational application that enables users to interact using natural language through one or more channels, such as email, web chat, voice messaging, or social media platform such as Microsoft Teams.
[img](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-qna-solution-qna-maker/media/bot-channels.png)

Question answering is often the starting point for bot development - particularly for conversational dialogs that involve answering user questions. For this reason, Language Studio provides the option to easily create a bot that runs in the Azure Bot Service based on your knowledge base.

To create a bot from your knowledge base, use Language Studio to deploy the bot and then use the *Create Bot* button to create a bot in your Azure subscription. You can then edit and customize your bot in the Azure portal.


