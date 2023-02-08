# Host Wordpress in ACI and Use AZ Storage for Volume Mounts

## Set Variables
RESOURCE_GROUP="AciWordpress"
LOCATION="eastus2"
STORAGE_ACCOUNT_NAME="acifileshareacct"
SQL_FILE_SHARE_NAME="acifileshare"
WP_SHARE_NAME_NAME="wpfileshare"
CONTAINER_GROUP_NAME="aciwordpress"
DNS_NAME_LABEL="aciwordpress" 

## Create Resource Group
``` az group create -n $RESOURCE_GROUP -l $LOCATION ```

## Create Storage Account
``` az storage account create --resource-group $RESOURCE_GROUP --name $STORAGE_ACCOUNT_NAME --location $LOCATION --sku Standard_LRS ```

## Create File Shares
``` az storage share create --name $SQL_FILE_SHARE_NAME --account-name $STORAGE_ACCOUNT_NAME ```
``` az storage share create --name $WP_SHARE_NAME_NAME --account-name $STORAGE_ACCOUNT_NAME ```

## Get Storage Account Key
``` STORAGE_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP --account-name $STORAGE_ACCOUNT_NAME --query "[0].value" --output tsv) ```

## Deploy Container Group ARM Template  
``` az group deployment create -n TestDeployment -g $RESOURCE_GROUP --template-file "aci-wordpress.json" --parameters 'mysqlPassword=My5q1P@s5w0rd!' --parameters "containerGroupName=$CONTAINER_GROUP_NAME" --parameters "dnsNameLabel=$DNS_NAME_LABEL" --parameters 'mysqlRootPassword=TedMy5q1P@s5w0rd!' --parameters "mysqlUser=wordpress" --parameters "mysqlUser=wordpress" --parameters "mysqlDbName=wordpress" --parameters "mysqlShareName=$SQL_FILE_SHARE_NAME" "wordpressShareName=$WP_SHARE_NAME_NAME" --parameters "storageAccountName=$STORAGE_ACCOUNT_NAME" --parameters "storageAccountKey=$STORAGE_KEY" ```

## Get Domain Name of WP Container 
``` az container show -g $RESOURCE_GROUP -n $CONTAINER_GROUP_NAME --query ipAddress.fqdn -o tsv ```