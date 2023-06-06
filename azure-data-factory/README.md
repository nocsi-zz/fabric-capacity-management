
# Fabric Capacity Management using Azure Data Factory (ADF)


## Prerequisites:
-	Add your own Azure Data Factory or use an existing one.
-	Add your own Microsoft Fabric Capacity in the Azure portal.
    - In the Access Control (IAM) menu add the ADFs managed identity as contributor, allowing ADF to modify the capacity.
-	Add the 4 pipelines in ADF from the JSON files in this repository.
    - Fill out the default parameters in the 4 pipelines with your Fabric capacity information. You will find the values on the Overview page of the capacity in the Azure portal. 


## Pipelines (API endpoints):
-	[Fabric Capacities](Fabric%20Capacities.json): Lists the information about capacity, like sku, state (paused/active), administration members etc.
-	[Fabric Capacity Resume](Fabric%20Capacity%20Resume.json): Resumes (starts) the Fabric capacity. The API is async but the ADF web activity is handling the retry logic and will only stop when Azure is done with the resume activity or if it fails.
-	[Fabric Capacity Suspend](Fabric%20Capacity%20Suspend.json): Suspends (pauses) the Fabric capacity. The API is async but the ADF web activity is handling the retry logic and will only stop when Azure is done with the suspend activity or if it fails.
-	[Fabric Capacity Sku](Fabric%20Capacity%20Sku.json): Updates the Fabric capacity to the sku defined in the parameter when calling the pipeline. 

## Pros: 
-	It is possible to use ADFs Managed Identity to give access to the Fabric Capacity, meaning no need for an app registration. If you don’t have access to create an App registration needed for authentication, then ADF would be a good starting point.
-	ADF allows pasting the JSON code found in this repository into the UI, while Fabric currently only has read-only JSON, meaning it is faster to just test using ADF. 

## Cons: 
-	Using another Azure service like ADF to handle the capacity management seems like a bad design if everything else is in Fabric. 
