
# UploadToQnaMaker

## Deploy

[![Deploy to Azure](https://azuredeploy.net/deploybutton.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCareyjmac%2Fazure-search-power-skills%2Fqna%2FQnA%2FQnAMaker%2Fazuredeploy.json)

## How to use

1. Deploy the ARM template using the button above.
1. When the deployment finishes, go to the "Outputs" and copy and paste the outputs into the associated variables for the Postman collection.
1. Copy data files into the deployed storage account into the blob container name that you specified in the Postman variables.
1. Create a knowledge base in your newly deployed QnAMaker instance at [https://www.qnamaker.ai/](https://www.qnamaker.ai/).
    1. Grab the id for the knowledge base from the url, it will look something like this "https://www.qnamaker.ai/Edit/KnowledgeBase?kbId=**KnowledgeBaseID**&tenantId=**TenantId**" where you want the **KnowledgeBaseID**.
    1. [Add an application setting](https://docs.microsoft.com/en-us/azure/azure-functions/functions-how-to-use-azure-function-app-settings#portal) for the deployed function instance called "KnowledgeBaseID" with the value of the knowledge id you just grabbed.
1. Run the Postman commands in order:
    1. Create Datasource
    1. Create Index
    1. Create Skillset
    1. Create Indexer
1. Monitor the indexer status using the Postman command "Get Indexer Status" until it shows as complete. 
1. You should now be able to query the 'qna-idx' for search results. To monitor for QnA upload completion, check the storage account for a queue called "upload-to-qna".  Once that queue has no values remaining, the QnA upload process will be complete. If any queue messages end up in a queue called "upload-to-qna-poison", then there is likely something wrong with the queue execution that needs to be investigated.
    1. I recommend that if you need to debug the custom skill or queue trigger, to attach an Application Insights service to the function instance and then monitor the function while it is executing by using the [Log streaming option](https://docs.microsoft.com/azure/azure-functions/functions-monitoring?tabs=cmd#built-in-log-streaming) in the portal.

## Data
The data I was using to test with [can be found here](https://github.com/JerryWei03/COVID-Q/tree/master/data/PDFs) if you want to reuse it.  Some of these files consistently fail upload to QnAMaker for reasons unknown to me. Make sure if we end up using this for the final demo that [we reference the author](https://github.com/JerryWei03/COVID-Q#citation) and verify the licensing as appropriate.

## TODO

1. Build/integrate frontend
1. Add website deployment to the ARM template once the frontend is built. [See here for example.](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-github-deploy)
1. [Have the function app use the included hosting plan](https://docs.microsoft.com/azure/azure-functions/functions-scale#app-service-plan) instead of needing to deploy a second one.
    1. I am unsure based on the parameters that QnA requires for the hosting plan if we can also tack on the function, so that will need to be investigated/tested. 
    1. You will likely also want to use the hosting plan for the frontend to be included with the sample so ideally it would host all 3 resources when it is all said and done.
    1. If we do this, then we can in theory support an unlimited timeout for the queue trigger.  That would need to be tweaked in the host.json file.
    1. Also make sure the function is set to AlwaysOn so that the custom skill works correctly, per the linked documentation.
1. Add cognitive services enrichment skills to skillset/indexer output field mappings/index.
1. Fix [potential race condition](./UploadToQnAMaker.cs#L105) in index status update code.
1. Make the knowledge base id task a bit simpler.
    1. My idea for this is that instead of requiring it be set on the function app settings, pass it as a header to the custom skill. That way the customer just needs to paste it into the Postman collection (or otherwise the frontend once that exists) and it can be passed a long to the custom skill/queue trigger exeuction that way.
1. Documentation
