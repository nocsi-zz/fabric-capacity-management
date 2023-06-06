
# Fabric Capacity Management using Fabric Data Pipeline


## Prerequisites:
-	Add a new Fabric workspace or use an existing one.
-	Add an app registration in the Azure Active Directory.
    - Copy the "Application (client) ID" and "Directory (tenant) ID" from the overview page for later use.
    - Create a new Client Secret on the "Certificates & secrets" page. Give it a random description and 24 months to expiration. Save the secret "value" (not the secret id) for later use.
-	Add your own Microsoft Fabric Capacity in the Azure portal.
    - In the Access Control (IAM) menu add the app (service principal) that you just created as contributor, allowing the app to modify the capacity.
-	Add the 4 pipelines in the Data Factory section of your Fabric workspace. 
    - Currently the JSON view of a data pipeline is read-only meaning you will have to manually create the parameters and web activities. See the guides at the bottom of this page.
    - After creating the pipelines, fill out the default parameters subsciptionId, resourceGroupName and dedicatedCapacityName in the 4 pipelines with your Fabric capacity information. You will find the values on the Overview page of the capacity in the Azure portal. Then fill out clientId and tenantId from your App registration. The parameter clientSecret is of the type SecureString and needs to be filled out at runtime.


## Pipelines (API endpoints):
The pipelines will start with a request to get a token and then use this token for the subsequent activity. 
-	[Fabric Capacities](Fabric%20Capacities.json): Lists the information about capacity, like sku, state (paused/active), administration members etc.
-	[Fabric Capacity Resume](Fabric%20Capacity%20Resume.json): Resumes (starts) the Fabric capacity. The API is async but the ADF web activity is handling the retry logic and will only stop when Azure is done with the resume activity or if it fails.
-	[Fabric Capacity Suspend](Fabric%20Capacity%20Suspend.json): Suspends (pauses) the Fabric capacity. The API is async but the ADF web activity is handling the retry logic and will only stop when Azure is done with the suspend activity or if it fails.
-	[Fabric Capacity Sku](Fabric%20Capacity%20Sku.json): Updates the Fabric capacity to the sku defined in the parameter when calling the pipeline. 

## Pros: 
-	Using Fabric Data Pipelines to handle the capacity management will work towards a strategy to handle all logic in Fabric, without the need for other Azure services.

## Cons: 
-	Currently there is no support for managed identity in Fabric meaning the hassle and vulnerability of using an app registration with a secret.
-	The need for the web activity to get the token in Fabric Data Pipelines. ADFs web activity has the possibility to use "service principal" for authentication through an app, which would make the pipeline simpler. Not to mention the possibility to use Azure KeyVault for referencing the App Secret in a secure place.



## Guide for Fabric Capacities pipeline:

| Description  | Image |
| ------------- | ------------- |
| Add the parameters | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/c0b271f4-d975-44fb-a260-b2fc9a38f591)  |
| Add the Variable "apiVersion" with the value "2022-07-01-preview". This could change in the future when it is out of preview.  | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/2de53dee-02ce-4875-b6cb-23fd5d2d8e67)  |
| Add two "Web" activities and connect them "On success". Give the first activity the name "App Fabric Capacity Token" and the second activity the name "Fabric Capacities" (optionally). | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/8d46d113-475c-412e-91c1-76f4bd3dbb32) |
| On the "App Fabric Capacity Token" activity set Secure output and input to true in the advanced section: | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/900c9aea-2521-4985-b0cc-4014095a3a71) |
| On the Settings tab create a new connection with a Base Url "https://login.microsoftonline.com" and a optional "Connection name": | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/4afa791f-0d37-402e-8bf6-4f1783992b9a) |
| The other settings:| ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/c0aceb33-207d-4819-b424-cdd2b02a514f) |
| - Relative URL (dynamic content): /@{pipeline().parameters.tenantId}/oauth2/token  ||
| - Method: POST ||
| - Body (dynamic content): client_id=@{pipeline().parameters.clientId}&client_secret=@{json(string(pipeline().parameters.clientSecret)).value}&grant_type=client_credentials&resource=https://management.azure.com ||
| - Headers (new): Content-Type = application/x-www-form-urlencoded  ||
| The other web activity ("Fabric Capacities") should have Secure input set to true in the advanced section: | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/0697430c-69b9-4024-8c98-3497a5f26324) |
| On the Settings tab create a new connection with a Base Url "https://login.microsoftonline.com" and a optional "Connection name": | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/d57c5e70-fa7f-4ff7-a8b0-2b96e983b487) |
| The other settings: | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/2e45a0a9-55bb-43cf-adbf-ff10c536f5ab) |
| - Relative URL (dynamic content): /subscriptions/@{pipeline().parameters.subsciptionId}/resourceGroups/@{pipeline().parameters.resourceGroupName}/providers/Microsoft.Fabric/capacities/@{pipeline().parameters.dedicatedCapacityName}?api-version=@{variables('apiVersion')} |  |
| - Method: GET |  |
| - Headers (new using dynamic content in value): Authorization = Bearer @{activity('App Fabric Capacity Token').output.access_token} |  |


Save and run. To debug compare the JSON code in the View->View JSON code with the code in [Fabric Capacities](Fabric%20Capacities.json).



## Guide for Fabric Capacity Resume pipeline:
Follow guide for Fabric Capacities pipeline but change the second web activity:
| Description  | Image |
| ------------- | ------------- |
| Change the name of the second web activity to "Fabric Capacity Resume" (optionally). | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/72258ec2-44cc-4741-9f93-fa82d319e853) |
| Change the settings to: | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/b6b66ebf-b7dd-4f79-b572-b6c00848641c) |
| - Relative URL (dynamic content): /subscriptions/@{pipeline().parameters.subsciptionId}/resourceGroups/@{pipeline().parameters.resourceGroupName}/providers/Microsoft.Fabric/capacities/@{pipeline().parameters.dedicatedCapacityName}/resume?api-version=@{variables('apiVersion')}  |  |
| - Method: POST |  |
| - Body (dummy input because POST method in web activity has to have an input...): {"dummy":true} |  |
| - Headers (new using dynamic content in value): Authorization = Bearer @{activity('App Fabric Capacity Token').output.access_token} |  |



## Guide for Fabric Capacity Suspend pipeline:
Follow guide for Fabric Capacities pipeline but change the second web activity:
| Description  | Image |
| ------------- | ------------- |
| Change the name of the second web activity to "Fabric Capacity Suspend" (optionally). | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/ef23abf4-0039-4988-bc37-c84980265d25) |
| Change the settings to: | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/d949bd62-7dfa-46a1-8d65-0a79a9e860f9) |
| - Relative URL (dynamic content): /subscriptions/@{pipeline().parameters.subsciptionId}/resourceGroups/@{pipeline().parameters.resourceGroupName}/providers/Microsoft.Fabric/capacities/@{pipeline().parameters.dedicatedCapacityName}/suspend?api-version=@{variables('apiVersion')}  |  |
| - Method: POST |  |
| - Body (dummy input because POST method in web activity has to have an input...): {"dummy":true} |  |
| - Headers (new using dynamic content in value): Authorization = Bearer @{activity('App Fabric Capacity Token').output.access_token} |  |




## Guide for Fabric Capacity Sku pipeline:
Follow guide for Fabric Capacities pipeline but change the second web activity:
| Description  | Image |
| ------------- | ------------- |
| Add pipeline parameter "sku" | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/c6358762-4ffa-4149-9f28-6506ada7408e) |
| Change the name of the second web activity to "Fabric Capacity Sku" (optionally). | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/488da801-b4b9-4fea-aaee-58a122e28824) |
| Change the settings to: | ![image](https://github.com/nocsi-zz/fabric-capacity-management/assets/1149028/df5da46d-8d0f-4637-8ef8-43da9f8a86de) |
| - Relative URL (dynamic content): /subscriptions/@{pipeline().parameters.subsciptionId}/resourceGroups/@{pipeline().parameters.resourceGroupName}/providers/Microsoft.Fabric/capacities/@{pipeline().parameters.dedicatedCapacityName}?api-version=@{variables('apiVersion')}  |  |
| - Method: PATCH |  |
| - Body (dynamic content): {  "sku": {    "name": "@{pipeline().parameters.sku}"  }} |  |
| - Header 1 (new using dynamic content in value): Authorization = Bearer @{activity('App Fabric Capacity Token').output.access_token} |  |
| - Header 2 (new, seems like a bug in Fabric Pipeline that we need this one): Content-Type = application/json |  |




