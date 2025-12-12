# TypeScript quickstart for classic RAG in Azure AI Search

Demonstrates how to use TypeScript and the [Azure SDK for JavaScript/TypeScript](https://learn.microsoft.com/javascript/api/overview/azure/search-documents-readme?view=azure-node-latest) to query an Azure AI Search index and send the results to a chat completion model that provides an answer.

This Node.js console application uses the [Azure SDK for JavaScript/TypeScript](https://www.npmjs.com/package/@azure/search-documents) and runs on a search service using connection information that you provide.

**NOTE:** We now recommend [agentic retrieval](https://learn.microsoft.com/azure/search/agentic-retrieval-overview) for RAG workflows, but classic RAG is simpler. If it meets your application requirements, it can still be a good choice.

## Prerequisites

+ [Azure AI Search](https://learn.microsoft.com/azure/search/search-create-service-portal), Basic pricing tier or higher, with the [hotels-sample-index](https://learn.microsoft.com/azure/search/search-get-started-portal). The sample index must have a [default semantic configuration](https://learn.microsoft.com/azure/search/search-get-started-rag&pivots=javascript#create-an-index).

+ [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/how-to/create-resource) with a deployment of a chat completion model, such as GPT-4.1 or GPT-4.1-mini.

+ [Visual Studio Code](https://code.visualstudio.com) or another IDE.

+ [Node.JS with LTS](https://nodejs.org/en/download/).

+ [TypeScript](https://www.typescriptlang.org/download/). We recommend a global installation: `npm install -g typescript`. You might also need `npm install @types/node` to support type definitions.

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
   cd quickstart-rag-ts
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

1. Open the quickstart-rag-ts folder and find the `tsconfig.json` file. This is the configuration file that specifies the source files.

1. Build the TypeScript code to JavaScript. Both `.ts` files in `src` now have `.js` equivalents in the `dist` folder.

   ```cmd
   tsc
   ```

1. Open the `dist` folder and notice the `query.js` file. This is the source code for this sample, compiled from TypeScript.

1. Run the code from the root folder. The `.env` is passed into the runtime using the `-r dotenv/config`.

   ```cmd
   node -r dotenv/config dist/query.js
   ```

1. You should get output consisting of recommendations for several hotels. This output is generated by the chat completion model, which grounds its answer using the search results from the query.

1. A second JavaScript file has a more complex prompt that adds more structure to the output. The extra information includes a hotel address, description, tags, and hotel room pricing.

   ```cmd
   node -r dotenv/config dist/queryComplex.js
   ```