# Quickstart: Classic RAG in Azure AI Search using REST APIs

In this quickstart, you send search results to a chat completion model to ground a conversational search experience over your indexed content on Azure AI Search. After setting up Azure OpenAI and Azure AI Search resources in the Azure portal, you run code to call the APIs.

This code is based on the [classic RAG pattern](https://learn.microsoft.com/azure/search/search-agentic-retrieval-concept) in Azure AI Search. This pattern uses a chat completion model to provide a response to a query posed to your content on Azure AI Search.

**NOTE:** We now recommend [agentic retrieval](https://learn.microsoft.com/azure/search/agentic-retrieval-overview) for RAG workflows, but classic RAG is simpler. If it meets your application requirements, it can still be a good choice.

## Prerequisites

+ An [Azure AI Search service](https://learn.microsoft.com/azure/search/search-create-service-portal) with [semantic ranker enabled](https://learn.microsoft.com/azure/search/semantic-how-to-enable-disable).

+ An [Azure OpenAI resource](https://learn.microsoft.com/azure/ai-services/openai/how-to/create-resource).

  + [Choose a region](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=global-standard%2Cstandard-chat-completions#global-standard-model-availability) that supports the chat completion model you want to use (gpt-4o, gpt-4o-mini, or an equivalent model).
  + [Deploy the chat completion model](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-openai) in Microsoft Foundry or [use another approach](https://learn.microsoft.com/azure/ai-services/openai/how-to/working-with-models).
 
+ A [supported Azure OpenAI chat completion model](https://learn.microsoft.com/azure/search/search-agentic-retrieval-how-to-create#supported-models). This sample uses `gpt-4.1-mini`.

+ [Visual Studio Code](https://code.visualstudio.com/download) with the [REST Client extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client).

## Sign in to Azure

You're using Microsoft Entra ID and role assignments for the connection. Make sure you're logged in to the same tenant and subscription as Azure AI Search and Azure OpenAI. You can use the Azure CLI on the command line to show current properties, change properties, and to log in. For more information, see [Connect without keys](https://learn.microsoft.com/azure/search/search-get-started-rbac). 

Run each of the following commands in sequence.

```
az account show

az account set --subscription <PUT YOUR SUBSCRIPTION ID HERE>

az login --tenant <PUT YOUR TENANT ID HERE>
```

You should now be logged in to Azure from your local device.

## Configure access

Requests to the search endpoint must be authenticated and authorized. You can use API keys or roles for this task. Keys are easier to start with, but roles are more secure. This quickstart assumes roles.

You're setting up two clients, so you need permissions on both resources.

Azure AI Search is receiving the query request from your local system. Assign yourself the **Search Index Data Reader** role assignment if the hotels sample index already exists. If it doesn't exist, assign yourself **Search Service Contributor** and **Search Index Data Contributor** roles so that you can create and query the index.

Azure OpenAI is receiving the query and the search results from your local system. Assign yourself the **Cognitive Services OpenAI User** role on Azure OpenAI.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Configure Azure AI Search for role-based access:

    1. In the Azure portal, find your Azure AI Search service.

    1. On the left menu, select **Settings** > **Keys**, and then select either **Role-based access control** or **Both**.

1. Assign roles:

    1. On the left menu, select **Access control (IAM)**.

    1. On Azure AI Search, select these roles to create, load, and query a search index, and then assign them to your Microsoft Entra ID user identity:

       - **Search Index Data Contributor**
       - **Search Service Contributor**

    1. On Azure OpenAI, select **Access control (IAM)** to assign this role to yourself on Azure OpenAI:

       - **Cognitive Services OpenAI User**

It can take several minutes for permissions to take effect.

## Get service endpoints and tokens

In the remaining sections, you set up API calls to Azure OpenAI and Azure AI Search. Get the service endpoints and tokens so that you can provide them as variables in your code.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. [Find your search service](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices).

1. On the **Overview** home page, copy the URL. An example endpoint might look like `https://example.search.windows.net`. 

1. [Find your Azure OpenAI service](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.CognitiveServices%2Faccounts).

1. On the **Overview** home page, select the link to view the endpoints. Copy the URL. An example endpoint might look like `https://example.openai.azure.com/`.

1. Get personal access tokens from the Azure CLI on a command prompt. Here are the commands for each resource:

   - `az account get-access-token --resource https://search.azure.com --query "accessToken" -o tsv`
   - `az account get-access-token --resource https://cognitiveservices.azure.com --query "accessToken" -o tsv`

## Prepare the sample index

This quickstart assumes the hotels-sample-index, which you can create using [this quickstart](https://learn.microsoft.com/azure/search/search-get-started-portal).

Once the index exists, use the **Edit JSON** action in the Azure portal to add this semantic configuration:

```json
"semantic":{
    "defaultConfiguration":"semantic-config",
    "configurations":[
        {
            "name":"semantic-config",
            "prioritizedFields":{
            "titleField":{
                "fieldName":"HotelName"
            },
            "prioritizedContentFields":[
                {
                    "fieldName":"Description"
                }
            ],
            "prioritizedKeywordsFields":[
                {
                    "fieldName":"Category"
                },
                {
                    "fieldName":"Tags"
                }
            ]
            }
        }
    ]
},
```

## Run the code

1. Open the `rag.rest` file in Visual Studio Code.

1. Replace the placeholder variables with valid values.

1. Save the file.

1. To test the connection, send your first request.

   ```http
   ### List existing indexes by name (verify the connection)
    GET  {{searchUrl}}/indexes?api-version=2025-11-01-preview&$select=name  HTTP/1.1
    Authorization: Bearer {{personalAccessToken}}
   ```

1. Select **Send Request**.

1. Output for this GET request should be a list of indexes. You should see the **hotels-sample-index** among them.

## Set up the query and chat thread

1. Set up a query request on the phrase *"Can you recommend a few hotels with complimentary breakfast?"*. This query uses semantic ranking to return relevant matches, even if the verbatim text isn't an exact match. Results are held in the **searchRequest** variable for reuse on the next request.

   ```http
   # @name searchRequest
    POST {{searchUrl}}/indexes/{{index-name}}/docs/search?api-version={{api-version}} HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer {{searchAccessToken}}
    
    {
      "search": "Can you recommend a few hotels with complimentary breakfast?",
      "queryType": "semantic",
      "semanticConfiguration": "semantic-config",
      "select": "Description,HotelName,Tags",
      "top": 5
    }
    
    ### 3 - Use search results in Azure OpenAI call to a chat completion model
    POST {{aoaiUrl}}/openai/deployments/{{aoaiGptDeployment}}/chat/completions?api-version=2024-08-01-preview HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer {{aoaiAccessToken}}
    
    {
      "messages": [
        {
          "role": "system", 
          "content": "You recommend hotels based on activities and amenities. Answer the query using only the search result. Answer in a friendly and concise manner. Answer ONLY with the facts provided. If there isn't enough information below, say you don't know."
        },
        {
          "role": "user",
          "content": "Based on the hotel search results, can you recommend hotels with breakfast? Here are all the hotels I found:\n\nHotel 1: {{searchRequest.response.body.value[0].HotelName}}\nDescription: {{searchRequest.response.body.value[0].Description}}\n\nHotel 2: {{searchRequest.response.body.value[1].HotelName}}\nDescription: {{searchRequest.response.body.value[1].Description}}\n\nHotel 3: {{searchRequest.response.body.value[2].HotelName}}\nDescription: {{searchRequest.response.body.value[2].Description}}\n\nHotel 4: {{searchRequest.response.body.value[3].HotelName}}\nDescription: {{searchRequest.response.body.value[3].Description}}\n\nHotel 5: {{searchRequest.response.body.value[4].HotelName}}\nDescription: {{searchRequest.response.body.value[4].Description}}\n\nPlease recommend which hotels offer breakfast based on their descriptions."
        }
      ],
      "max_tokens": 1000,
      "temperature": 0.7
    }`
    ```

1. **Send** the request.

1. Output should look similar to the following example:

   ```json
      "value": [
        {
          "@search.score": 3.9269178,
          "@search.rerankerScore": 2.380699872970581,
          "HotelName": "Head Wind Resort",
          "Description": "The best of old town hospitality combined with views of the river and cool breezes off the prairie. Our penthouse suites offer views for miles and the rooftop plaza is open to all guests from sunset to 10 p.m. Enjoy a complimentary continental breakfast in the lobby, and free Wi-Fi throughout the hotel.",
          "Tags": [
            "coffee in lobby",
            "free wifi",
            "view"
          ]
        },
        {
          "@search.score": 1.5450059,
          "@search.rerankerScore": 2.1258809566497803,
          "HotelName": "Thunderbird Motel",
          "Description": "Book Now & Save. Clean, Comfortable rooms at the lowest price. Enjoy complimentary coffee and tea in common areas.",
          "Tags": [
            "coffee in lobby",
            "free parking",
            "free wifi"
          ]
        },
        {
          "@search.score": 2.2158256,
          "@search.rerankerScore": 2.121671438217163,
          "HotelName": "Swan Bird Lake Inn",
          "Description": "We serve a continental-style breakfast each morning, featuring a variety of food and drinks. Our locally made, oh-so-soft, caramel cinnamon rolls are a favorite with our guests. Other breakfast items include coffee, orange juice, milk, cereal, instant oatmeal, bagels, and muffins.",
          "Tags": [
            "continental breakfast",
            "free wifi",
            "24-hour front desk service"
          ]
        },
        {
          "@search.score": 0.6395861,
          "@search.rerankerScore": 2.116753339767456,
          "HotelName": "Waterfront Scottish Inn",
          "Description": "Newly Redesigned Rooms & airport shuttle. Minutes from the airport, enjoy lakeside amenities, a resort-style pool & stylish new guestrooms with Internet TVs.",
          "Tags": [
            "24-hour front desk service",
            "continental breakfast",
            "free wifi"
          ]
        },
        {
          "@search.score": 4.885111,
          "@search.rerankerScore": 2.0008862018585205,
          "HotelName": "Double Sanctuary Resort",
          "Description": "5 star Luxury Hotel - Biggest Rooms in the city. #1 Hotel in the area listed by Traveler magazine. Free WiFi, Flexible check in/out, Fitness Center & espresso in room.",
          "Tags": [
            "view",
            "pool",
            "restaurant",
            "bar",
            "continental breakfast"
          ]
        }
      ]
    ```

1. Set up a conversation turn with a chat completion model. This request includes a prompt that provides instructions for the response. The `max_tokens` value is large enough to accommodate the search results from the previous query.

   ```http
    POST {{aoaiUrl}}/openai/deployments/{{aoaiGptDeployment}}/chat/completions?api-version=2024-08-01-preview HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer {{aoaiAccessToken}}
    
    {
    "messages": [
    {
      "role": "system", 
      "content": "You  are a friendly assistant that recommends hotels based on activities and amenities. Answer the query using only the search result. Answer in a friendly and concise manner. Answer ONLY with the facts provided. If there isn't enough information below, say you don't know."
        },
    {
      "role": "user",
      "content": "Based on the hotel search results, can you recommend hotels with breakfast? Here are all the hotels I found:\n\nHotel 1: {{searchRequest.response.body.value[0].HotelName}}\nDescription: {{searchRequest.response.body.value[0].Description}}\n\nHotel 2: {{searchRequest.response.body.value[1].HotelName}}\nDescription: {{searchRequest.response.body.value[1].Description}}\n\nHotel 3: {{searchRequest.response.body.value[2].HotelName}}\nDescription: {{searchRequest.response.body.value[2].Description}}\n\nHotel 4: {{searchRequest.response.body.value[3].HotelName}}\nDescription: {{searchRequest.response.body.value[3].Description}}\n\nHotel 5: {{searchRequest.response.body.value[4].HotelName}}\nDescription: {{searchRequest.response.body.value[4].Description}}\n\nPlease recommend which hotels offer breakfast based on their descriptions."
    }
    ],
    "max_tokens": 1000,
    "temperature": 0.7
    }
    ```

1. **Send** the request.

1. Output should be an HTTP 200 Success status message. Included in the output is content that answers the question:

   ```json
    "message": {
      "annotations": [],
      "content": "I recommend the following hotels that offer breakfast:\n\n1. **Head Wind Resort** - Offers a complimentary continental breakfast in the lobby.\n2. **Swan Bird Lake Inn** - Serves a continental-style breakfast each morning, including a variety of food and drinks. \n\nEnjoy your stay!",
      "refusal": null,
      "role": "assistant"
    }
    ```

Notice that the output is missing several hotels that mention breakfast in the Tags field. The Tags field is an array, and including this field breaks the JSON structure in the results. Because there are no string conversion capabilities in the REST client, extra code for manually converting the JSON to a string is required if arrays are to be included. We omit this step for this quickstart.

## Next step

You can learn more about Azure AI Search on the [official documentation site](https://learn.microsoft.com/azure/search).
