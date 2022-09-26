## Azure Monitor Integration with Container Apps

For this example we will be creating an Container App Environment with Azure Monitor Enabled and then create a Diagnostic Settings resource to export those logs to a Azure Storage Account.

#### Prerequisite Resources

First create a storage account we will later export logs into this storage account using Azure Monitor
```bash
MY_RG=<MY_RG_NAME>
ACA_STORAGE_ACC=<STORAGE_ACC_NAME>
az storage account create -n $ACA_STORAGE_ACC -g $MY_RG --location australiaeast --sku Standard_LRS
```

#### Create a Container App Environment with Azure Monitor as the logging destination
```bash
ACA_ENV_NAME=<MY_ENV_NAME>
az deployment group create -n mycontainerappenv -g $MY_RG -f container-app-env-arm.json -p environment_name=$ACA_ENV_NAME
```

#### Create Corresponding Azure Monitor Diagnostic Settings Resource
```bash
# Get Container App Environment ID
ACA_ENV_ID=$(az containerapp env show -g $MY_RG -n $ACA_ENV_NAME --query id -o tsv)

# Get Storage Account Id
STORAGE_ACC_ID=$(az storage account show -n $ACA_STORAGE_ACC -g $MY_RG --query id -o tsv)

# Create Azure Monitor Resource
az monitor diagnostic-settings create --name mydiagnosticsettings --resource $ACA_ENV_ID --storage-account $STORAGE_ACC_ID --logs '@azure-monitor-log-options.json'
```

#### Produce Logs
```bash
# Create a Container App
az deployment group create -n mycontainerapp -g $MY_RG -f container-app-arm.json -p environment_name=$ACA_ENV_NAME
```

#### Check Log Destinations
Navigate to Storage Account in Portal and see exported logs
