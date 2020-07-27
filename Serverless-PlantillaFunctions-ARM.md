
AZURE FUNCIONS CON PLANTILLA ARM DE RECURSOS
-----------------------------------------------------------------

Ejemplo de plantilla ARM para crear Azure Functions

```
#script con plantilla ARM original 
#read -p "Enter a resource group name that is used for generating resource names:" resourceGroupName &&
#read -p "Enter the location (like 'eastus' or 'northeurope'):" location &&
#templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json" &&
#az group create --name $resourceGroupName --location "$location" &&
#az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
#echo "Press [ENTER] to continue ..." &&
#read

#script paso a paso para ejecutar la plantilla ARM
$ read -p "Enter a resource group name that is used for generating resource names:" resourceGroupName 
$ read -p "Enter the location (like 'eastus' or 'northeurope'):" location 
$ templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json"
$ az group create --name $resourceGroupName --location "$location" 
$ az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri 


#crear funcion paso a paso
tutorial: https://docs.microsoft.com/es-es/azure/azure-functions/functions-run-local

$ curl --get http://localhost:7071/api/MyHttpTrigger?name=Azure%20Rocks
$ curl --request POST http://localhost:7071/api/MyHttpTrigger --data '{"name":"Azure Rocks"}'

```
