
# Fabric Capacity Management using Fabric Data Pipeline


## Prerequisites:
-	Add a new Fabric workspace or use an existing one.
-	Add an app registration in the Azure Active Directory.
    - Copy the "Application (client) ID" and "Directory (tenant) ID" from the overview page for later use.
    - Create a new Client Secret on the "Certificates & secrets" page. Give it a random description and 24 months to expiration. Save the secret "value" (not the secret id) for later use.
-	Add your own Microsoft Fabric Capacity in the Azure portal.
    - In the Access Control (IAM) menu add the app (service principal) that you just created as contributor, allowing the app to modify the capacity.
-	Add the 4 pipelines in the Data Factory section of your Fabric workspace. 
    - Currently the JSON view of a data pipeline is read-only meaning you will have to manually create the parameters and web activities. See the guide in the bottom of this page.
    - After creating the pipelines, fill out the default parameters subsciptionId, resourceGroupName and dedicatedCapacityName in the 4 pipelines with your Fabric capacity information. You will find the values on the Overview page of the capacity in the Azure portal. Then fill out clientId and tenantId from your App registration. The parameter clientSecret is of the type SecureString and needs to be filled out at runtime.


## Pipelines (API endpoints):
The pipelines will start with a request to get a token and then use this token for the subsequent activity. 
-	[Fabric Capacities](Fabric%20Capacities.json): Lists the information about capacity, like sku, state (paused/active), administration members etc.
-	[Fabric Capacity Resume](Fabric%20Capacity%20Resume.json): Resumes (starts) the Fabric capacity. The API is async but the ADF web activity is handling the retry logic and will only stop when Azure is done with the resume activity or if it fails.
-	[Fabric Capacity Suspend](Fabric%20Capacity%20Suspend.json): Suspends (pauses) the Fabric capacity. The API is async but the ADF web activity is handling the retry logic and will only stop when Azure is done with the suspend activity or if it fails.
-	[Fabric Capacity Sku](Fabric%20Capacity%20Sku.json): Updates the Fabric capacity to the sku defined in the parameter when calling the pipeline. 

## Pros: 
-	Using Fabric Data Pipelines to handle the capacity management will work towards a strategy to handle all logic in Fabric, without the need for other Azure services.
-	It is possible to use ADFs Managed Identity to give access to the Fabric Capacity, meaning no need for an app registration. If you don’t have access to create an App registration needed for authentication, then ADF would be a good starting point.
-	ADF allows pasting the JSON code found in this repository into the UI, while Fabric currently only has read-only JSON, meaning it is faster to just test using ADF. 

## Cons: 
-	Currently there is no support for managed identity in Fabric meaning the hassle and vulnerability of using an app registration with a secret.
-	The need for the web activity to get the token in Fabric Data Pipelines. ADFs web activity has the possibility to use "service principal" for authentication through an app, which would make the pipeline simpler. Not to mention the possibility to use Azure KeyVault for referencing the App Secret in a secure place.

