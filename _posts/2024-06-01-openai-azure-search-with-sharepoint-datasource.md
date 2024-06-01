---
title: "OpenAI Azure AI Search with SharePoint Data Source"
last_modified_at: 2024-06-01T16:20:02-05:00
categories:
  - Blog
tags:
  - Microsoft Copilot
  - Copilot Studio
  - OpenAI
  - Azure AI Search
  - Search Index
  - SharePoint data source
---

In this post, we will show how to index SharePoint documents with Azure AI Search index and use it with an Azure Open AI model for chat playground. Curently, SharePoint data source is not available in the Azure AI Search UI but we can configure using REST API. This GPT model further can be integrated with Copilot Studio and deployed to various channels such as MS Teams.

## Pre-Requisites

- SharePoint sites with documents
- Azure Entra Service Principal
- Azure AI Search (to create SharePoint as Data Source, Search Index and Indexer)
- Open AI Access

## Create Service Principal 

We need to create an Azure Entra ID service principal to configure the SharePoint data source. In this case, Azure AI Search uses Graph API to access the SharePoint data. we use delegated permissions to configure app registration. You can can also use application permissions. You need to create client secret if you are using application permissions.

Navigate to Azure Entra Id -> App Registrations -> Create App Registration

Select API Permissions blade -> Add a permission -> Select Microsoft Graph -> Delegated Permissions -> Select Files.Read and Sites.Read permissions

Select Authentication blade -> Add a platform -> Select 'Mobile and desktop applications' -> Select Redirect URIs and set 'Allow public client flows' to Yes

## Create Azure AI Search resources

Now, we need to create Azure AI Search resource, SharePoint data source, search index and indexer using the REST API or Power Shell as SharePoint data source is not available in the Azure UI. Both PowerShell and Postman scripts ate provided to create each resource. Use either PowerShell or Postman script as per your toolset 

**Create Azure AI Search resource**

In Azure Portal, Search for 'AI Search' ->  Create a search service -> Select a Resource Group -> Enter Service Name -> Create

Search service should be created with the URL: https://<searchservicename>.search.windows.net

Navigate to Keys blade and grab Primary admin key. This will be your apikey later used to create further resources

**Create SharePoint Data Source**

Run the below powershell script to create a SharePoint data source in AI Search. In this case. we are querying the data from a specifid document librray. you can use the entore site url as well. Replace variables as per your Search Service and SharePoint site URL. 
Include ApplicationSecret=<clientsecret> in connectionString in case of application permissions

```powershell
$headers = @{ "api-key" = "<apikey>" }

$request = @"
{
    "name" : "sharepoint-datasource",
    "type" : "sharepoint",
    "credentials" : { 
        "connectionString" : "SharePointOnlineEndpoint=https://tenant.sharepoint.com/sites/sitename;ApplicationId=<appid>;TenantId=<tenantid>"
    },
    "container" : { 
        "name" : "useQuery", 
        "query" : "includeLibrary=https://tenant.sharepoint.com/sites/sitename/documentlibrary"
    }
}
"@

$uri = "https://<searchservicename>.search.windows.net/datasources?api-version=2023-10-01-Preview"

Invoke-RestMethod -Uri $uri -Headers $headers -Method Post -Body $request -ContentType "application/json"        
```

```postman
POST https://<searchservicename>.search.windows.net/datasources?api-version=2023-10-01-Preview

Content-Type: application/json
api-key: <apikey>

{
    "name" : "sharepoint-datasource",
    "type" : "sharepoint",
    "credentials" : { "connectionString" : "SharePointOnlineEndpoint=https://tenant.sharepoint.com/sites/sitename;ApplicationId=<appid>;TenantId=<tenantid>" },
    "container" : { "name" : "useQuery", "query" : "includeLibrary=https://tenant.sharepoint.com/sites/sitename/documentlibrary" }
}
```

**Create SharePoint Index**

Run below Power Shell script to create SharePoint search index. Replace variables as per your tenant resources

```powershell
$headers = @{ "api-key" = "<apikey>" }

$request = @"
{
    "name" : "sharepoint-index",
    "fields":  [
        { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
        { "name": "metadata_spo_item_name", "type": "Edm.String", "key": false, "searchable": true, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_spo_item_path", "type": "Edm.String", "key": false, "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_spo_item_content_type", "type": "Edm.String", "key": false, "searchable": false, "filterable": true, "sortable": false, "facetable": true },
        { "name": "metadata_spo_item_last_modified", "type": "Edm.DateTimeOffset", "key": false, "searchable": false, "filterable": false, "sortable": true, "facetable": false },
        { "name": "metadata_spo_item_size", "type": "Edm.Int64", "key": false, "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
    ]
}
"@

$uri = "https://<searchservicename>.search.windows.net/indexes?api-version=2023-10-01-Preview"

Invoke-RestMethod -Uri $uri -Headers $headers -Method Post -Body $request -ContentType "application/json"      
```

```postman
POST https://<searchservicename>.search.windows.net/indexes?api-version=2023-10-01-Preview

Content-Type: application/json
api-key: <apikey>

{
    "name" : "sharepoint-index",
    "fields": [
        { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
        { "name": "metadata_spo_item_name", "type": "Edm.String", "key": false, "searchable": true, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_spo_item_path", "type": "Edm.String", "key": false, "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_spo_item_content_type", "type": "Edm.String", "key": false, "searchable": false, "filterable": true, "sortable": false, "facetable": true },
        { "name": "metadata_spo_item_last_modified", "type": "Edm.DateTimeOffset", "key": false, "searchable": false, "filterable": false, "sortable": true, "facetable": false },
        { "name": "metadata_spo_item_size", "type": "Edm.Int64", "key": false, "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
    ]
}
```

**Create Search Indexer**

Run below Power Shell script to create search indexer. Replace variables as per your tenant resources. We need to execute the next script within 10 minutes after executing the below script and complete the device login and the indexer script. Indexer script waits until we complete the devicelogin.

```powershell
$headers = @{ "api-key" = "<apikey>" }

$request = @"
{
    "name" : "sharepoint-indexer",
    "dataSourceName" : "sharepoint-datasource",
    "targetIndexName" : "sharepoint-index",
    "parameters": {
    "batchSize": null,
    "maxFailedItems": null,
    "maxFailedItemsPerBatch": null,
    "base64EncodeKeys": null,
    "configuration": {
        "indexedFileNameExtensions" : ".pdf, .docx, .pptx",
        "excludedFileNameExtensions" : ".png, .jpg, .gif",
        "dataToExtract": "contentAndMetadata"
      }
    },
    "schedule" : {},
    "fieldMappings" : [
        { 
          "sourceFieldName" : "metadata_spo_site_library_item_id", 
          "targetFieldName" : "id", 
          "mappingFunction" : { 
            "name" : "base64Encode" 
          } 
        }
    ]
}
"@


$uri = "https://<searchservicename>.search.windows.net/indexers?api-version=2023-10-01-Preview"

Invoke-RestMethod -Uri $uri -Headers $headers -Method Post -Body $request -ContentType "application/json"        
```

```postman
POST https://<searchservicename>.search.windows.net/indexers?api-version=2023-10-01-Preview
Content-Type: application/json
api-key: <apikey>

{
    "name" : "sharepoint-indexer",
    "dataSourceName" : "sharepoint-datasource",
    "targetIndexName" : "sharepoint-index",
    "parameters": {
    "batchSize": null,
    "maxFailedItems": null,
    "maxFailedItemsPerBatch": null,
    "base64EncodeKeys": null,
    "configuration": {
        "indexedFileNameExtensions" : ".pdf, .docx",
        "excludedFileNameExtensions" : ".png, .jpg",
        "dataToExtract": "contentAndMetadata"
      }
    },
    "schedule" : { },
    "fieldMappings" : [
        { 
          "sourceFieldName" : "metadata_spo_site_library_item_id", 
          "targetFieldName" : "id", 
          "mappingFunction" : { 
            "name" : "base64Encode" 
          } 
         }
    ]
}
```

**Get Search Indexer Status**

Execute the below script while indexer script awaits to complete tje device login. The below script returns a response with errorMessage and the code to authenticate.

```powershell
$headers = @{ "api-key" = "<apikey>" }

$uri = "https://<searchservicename>.search.windows.net/indexers/sharepoint-indexer/status?api-version=2023-10-01-Preview"

Invoke-RestMethod -Uri $uri -Headers $headers -Method Post -Body $request -ContentType "application/json"   
```

```postman
  GET https://<searchservicename>.search.windows.net/indexers/sharepoint-indexer/status?api-version=2023-10-01-Preview

  Content-Type: application/json
  api-key: [apikey]
```

Navigate to https://microsoft.com/devicelogin and enter the code returned from the above script and approve permissions. It will ask you to signin into the tenant. 

The Search Indexer will access the SharePoint content as the signed-in user. The user that logs in during this step will be that signed-in user. So, if you sign in with a user account that doesn’t have access to a document in the Document Library that you want to index, the indexer won’t have access to that document.

Now, Search Indexer should be able to index all of the documents the signed in user has access to within the SharePoint document library.

## Create OpenAI Service ## 

Search 'Azure OpenAI' in Azure Portal -> Select Resource group, Name and create a OpenAI resource

Navigate to Azure OpenAI Studio from OpenAI resource

Select 'Bring your own data' -> Select Azure AI Seach frm data source dropdown -> Select Azure AI Search Service and Search Index created above

Select Search type 'Keyword' -> Save and close

Start chatting with your SharePoint data in the Azure Open AI Chat playground

## Deploy to Copilot Studio ## 

You can fine tune and test the model and further deploy to Copilot Stuidio from Azure Open AI Chat playgroud and access your copilot from Copilot Studio. 







