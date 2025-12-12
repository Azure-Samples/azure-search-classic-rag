# JavaScript quickstart for classic RAG in Azure AI Search

Demonstrates how to use JavaScript and the [Azure SDK for JavaScript/TypeScript](https://learn.microsoft.com/javascript/api/overview/azure/search-documents-readme?view=azure-node-latest) to query an Azure AI Search index and send the results to a chat completion model that provides an answer.

This Node.js console application uses the [Azure SDK for JavaScript/TypeScript](https://www.npmjs.com/package/@azure/search-documents) and runs on a search service using connection information that you provide.

**NOTE:** We now recommend [agentic retrieval](https://learn.microsoft.com/azure/search/agentic-retrieval-overview) for RAG workflows, but classic RAG is simpler. If it meets your application requirements, it can still be a good choice.

## Prerequisites

+ [Azure AI Search](https://learn.microsoft.com/azure/search/search-create-service-portal), Basic pricing tier or higher, with the [hotels-sample-index](https://learn.microsoft.com/azure/search/search-get-started-portal). The sample index must have a [default semantic configuration](https://learn.microsoft.com/azure/search/search-get-started-rag&pivots=javascript#create-an-index).

+ [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/how-to/create-resource) with a deployment of a chat completion model, such as GPT-4.1 or GPT-4.1-mini.

+ [Visual Studio Code](https://code.visualstudio.com) or another IDE.

+ [Node.JS with LTS](https://nodejs.org/en/download/).

## Sign in to Azure

You're using Microsoft Entra ID and role assignments for the connection. Make sure you're signed in to the same tenant and subscription as Azure AI Search and Azure OpenAI. You can use the Azure CLI on the command line to show current properties, change properties, and to sign in. For more information, see [Connect without keys](https://learn.microsoft.com/azure/search/search-get-started-rbac). 

Run each of the following commands in sequence.

```azure-cli
az account show

az account set --subscription <PUT YOUR SUBSCRIPTION ID HERE>

az login --tenant <PUT YOUR TENANT ID HERE>
```

You're now signed in to Azure from your local device.

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

## Get service endpoints

In the remaining sections, you set up API calls to Azure OpenAI and Azure AI Search. Get the service endpoints so that you can provide them as variables in your code.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. [Find your search service](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices).

1. On the **Overview** home page, copy the URL. An example endpoint might look like `https://example.search.windows.net`. 

1. [Find your Azure OpenAI service](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.CognitiveServices%2Faccounts).

1. On the **Overview** home page, select the link to view the endpoints. Copy the URL. An example endpoint might look like `https://example.openai.azure.com/`.

## Prepare the sample index

This quickstart assumes the hotels-sample-index, which you can create using [this quickstart](https://learn.microsoft.com/azure/search/search-get-started-portal).

1. Once the index exists, use the **Edit JSON** action in the Azure portal to add this semantic configuration:

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

1. **Save** your changes.

## Set up the sample

1. Clone or download this sample repository.

1. Open the folder in Visual Studio Code and open a terminal window for command line operations.

1. Navigate to the quickstart folder:

   ```cmd
   cd quickstart-rag-js
   ```

1. Create a new package for ESM modules in your project directory. You should have `package-lock.json` and `package.json` when you complete this step.

    ```cmd
    npm init -y
    npm pkg set type=module
    ```

1. Edit the file `sample.env`, adding the connection information that's valid for your Azure resources, and then saving is as `.env`.

   ```cmd
    AZURE_SEARCH_ENDPOINT=YOUR-SEARCH-SERVICE-ENDPOINT-GOES-HERE
    AZURE_SEARCH_INDEX_NAME=hotels-sample-index
    
    AZURE_OPENAI_ENDPOINT=YOUR-AZURE-OPENAI-ENDPOINT-GOES-HERE
    AZURE_OPENAI_VERSION=2024-04-01-preview
    AZURE_DEPLOYMENT_MODEL=gpt-4o-mini
   ```

1. Install the packages used in this sample. You should see a `node_modules` folder when you complete this step.

    ```cmd
    npm install @azure/identity @azure/search-documents openai dotenv
    ```

## Run the sample

1. Open the `src` folder and notice the `query.js` file. This is the source code for this sample.

    It does the following:
    - Imports the necessary libraries for Azure AI Search and Azure OpenAI.
    - Uses environment variables to configure the Azure AI Search and Azure OpenAI clients.
    - Defines a function to get the clients for Azure AI Search and Azure OpenAI, using environment variables for configuration.
    - Defines a function to query Azure AI Search for sources based on the user query.
    - Defines a function to query Azure OpenAI for a response based on the user query and the sources retrieved from Azure AI Search.
    - The `main` function orchestrates the flow by calling the search and OpenAI functions, and then prints the response. 

1. Compile and run the code from the root folder. The `.env` is passed into the runtime using the `-r dotenv/config`.

   ```cmd
   node -r dotenv/config query.js
   ```

1. You should get output consisting of recommendations for several hotels. This output is generated by the chat completion model, which grounds its answer using the search results from the query. Here's an example of what the output might look like:
    
    ```
    Sure! Here are a few hotels that offer complimentary breakfast:
    
    - **Head Wind Resort**
    - Complimentary continental breakfast in the lobby
    - Free Wi-Fi throughout the hotel
    
    - **Double Sanctuary Resort**
    - Continental breakfast included
    
    - **White Mountain Lodge & Suites**
    - Continental breakfast available
    
    - **Swan Bird Lake Inn**
    - Continental-style breakfast each morning with a variety of food and drinks 
        such as caramel cinnamon rolls, coffee, orange juice, milk, cereal, 
        instant oatmeal, bagels, and muffins
    ```

If you get a **Forbidden** error message, check Azure AI Search configuration to make sure role-based access is enabled.

If you get an **Authorization failed** error message, wait a few minutes and try again. It can take several minutes for role assignments to become operational.

If you get a **Resource not found** error message, check the resource URIs and make sure the API version on the chat model is valid.

Otherwise, to experiment further, change the query and rerun the last step to better understand how the model works with the grounding data.

You can also modify the prompt to change the tone or structure of the output.

You might also try the query without semantic ranking by setting `use_semantic_reranker=False` in the query parameters step. Semantic ranking can noticably improve the relevance of query results and the ability of the LLM to return useful information. Experimentation can help you decide whether it makes a difference for your content.

## Send a complex RAG query

Azure AI Search supports [complex types](../../search-howto-complex-data-types.md) for nested JSON structures. In the hotels-sample-index, `Address` is an example of a complex type, consisting of `Address.StreetAddress`, `Address.City`, `Address.StateProvince`, `Address.PostalCode`, and `Address.Country`. The index also has complex collection of `Rooms` for each hotel.

If your index has complex types, change your prompt to include formatting instructions: 

```text
Can you recommend a few hotels that offer complimentary breakfast? 
Tell me their description, address, tags, and the rate for one room that sleeps 4 people.
```

1. Open the `queryComplex.js` file to view the code. This second JavaScript file has a more complex prompt that adds more structure to the output. The extra information includes a hotel address, description, tags, and hotel room pricing.


1. Run the following command in a terminal to execute the query script:

    ```bash
    node -r dotenv/config queryComplex.js
    ```

    The `.env` is passed into the runtime using the `-r dotenv/config`. 

1. View the output from Azure OpenAI, and it adds content from complex types.
    
    ```
    Here are a few hotels that offer complimentary breakfast and have rooms that sleep 4 people:
    
    1. **Head Wind Resort**
       - **Description:** The best of old town hospitality combined with views of the river and 
       cool breezes off the prairie. Enjoy a complimentary continental breakfast in the lobby, 
       and free Wi-Fi throughout the hotel.
       - **Address:** 7633 E 63rd Pl, Tulsa, OK 74133, USA
       - **Tags:** Coffee in lobby, free Wi-Fi, view
       - **Room for 4:** Suite, 2 Queen Beds (Amenities) - $254.99
    
    2. **Double Sanctuary Resort**
       - **Description:** 5-star Luxury Hotel - Biggest Rooms in the city. #1 Hotel in the area 
       listed by Traveler magazine. Free WiFi, Flexible check in/out, Fitness Center & espresso 
       in room. Offers continental breakfast.
       - **Address:** 2211 Elliott Ave, Seattle, WA 98121, USA
       - **Tags:** View, pool, restaurant, bar, continental breakfast
       - **Room for 4:** Suite, 2 Queen Beds (Amenities) - $254.99
    
    3. **Swan Bird Lake Inn**
       - **Description:** Continental-style breakfast featuring a variety of food and drinks. 
       Locally made caramel cinnamon rolls are a favorite.
       - **Address:** 1 Memorial Dr, Cambridge, MA 02142, USA
       - **Tags:** Continental breakfast, free Wi-Fi, 24-hour front desk service
       - **Room for 4:** Budget Room, 2 Queen Beds (City View) - $85.99
    
    4. **Gastronomic Landscape Hotel**
       - **Description:** Known for its culinary excellence under the management of William Dough, 
       offers continental breakfast.
       - **Address:** 3393 Peachtree Rd, Atlanta, GA 30326, USA
       - **Tags:** Restaurant, bar, continental breakfast
       - **Room for 4:** Budget Room, 2 Queen Beds (Amenities) - $66.99
    ...
       - **Tags:** Pool, continental breakfast, free parking
       - **Room for 4:** Budget Room, 2 Queen Beds (Amenities) - $60.99
    
    Enjoy your stay! Let me know if you need any more information.
    ```

## Troubleshooting errors

To debug Azure SDK errors, set the environment variable `AZURE_LOG_LEVEL` to one of the following: `verbose`, `info`, `warning`, `error`. This will enable detailed logging for the Azure SDK, which can help identify [issues with authentication](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/identity/identity/TROUBLESHOOTING.md#enable-and-configure-logging), network connectivity, or other problems.

Rerun the query script. You should now get informational statements from the SDKs in the output that provide more detail about any issues.

If you see output messages related to ManagedIdentityCredential and token acquisition failures, it could be that you have multiple tenants, and your Azure sign-in is using a tenant that doesn't have your search service. To get your tenant ID, search the Azure portal for "tenant properties" or run `az login tenant list`.

Once you have your tenant ID, run `az login --tenant <YOUR-TENANT-ID>` at a command prompt, and then rerun the script.
