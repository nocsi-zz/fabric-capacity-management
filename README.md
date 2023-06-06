# Code snippets and guides to use Microsoft Fabric Capacity Management 

Do you need to automate scaling of Microsoft Fabric capacities or maybe pausing the capacity at night? Then take a look at the following helper code snippets.
The code shows how to use the Microsoft Fabric Capacity REST API in [Fabric Pipelines](/fabric-data-pipelines), [Azure Data Factory](/azure-data-factory) and [Postman](/postman).

The use cases for setting up capacity management could be:
-	Scale up a Fabric capacity at night for a data load and then scale down again, if the end-users in PowerBI is satisfied with a smaller capacity.
-	Use a multi capacity environment having the orchestration on a small F2 capacity and then use this capacity to start/stop other capacities when needed.
-	Having a development environment on a designated Fabric capacity with a scheduled pipeline that pauses the capacity at night.

