
# Fabric Capacity Management using Postman


## Prerequisites:
-	Add an app registration in the Azure Active Directory.
    - Copy the "Application (client) ID" and "Directory (tenant) ID" from the overview page for later use.
    - Create a new Client Secret on the "Certificates & secrets" page. Give it a random description and 24 months to expiration. Save the secret "value" (not the secret id) for later use.
-	Add your own Microsoft Fabric Capacity in the Azure portal.
    - In the Access Control (IAM) menu add the app (service principal) that you just created as contributor, allowing the app to modify the capacity.
-	Download and install Postman (https://www.postman.com/downloads/). 
    - Import the [Postman collection](fabric-capacity-management-postman-collection.json).
    - Fill out the "Variables" tab with subsciptionId, resourceGroupName and dedicatedCapacityName from the Overview page of the capacity in the Azure portal. Then fill out clientId, clientSecret and tenantId from your App registration.
    - In the Authorization tab "Get new access token".


## Requests (API endpoints):
-	Fabric Capacities: Lists the information about capacity, like sku, state (paused/active), administration members etc.
-	Fabric Capacity Resume: Resumes (starts) the Fabric capacity. The API is async meaning your will get an 202 Accepted.
-	Fabric Capacity Suspend: Suspends (pauses) the Fabric capacity. The API is async meaning your will get an 202 Accepted.
-	Fabric Capacity Sku: Updates the Fabric capacity to the sku defined in the Body when calling the API. 
