.. _tf_storage:

###################
Terraform Storage
###################

Setup
=============

Setup Storage Account for Statefile

https://learn.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=terraform

1. Configure remote state storage account
2. Configure terraform backend state
   To configure the backend state, you need the following Azure storage information:

   * storage_account_name: The name of the Azure Storage account.
   * container_name: The name of the blob container.
   * key: The name of the state store file to be created.
   * access_key: The storage access key.

.. code-block:: bash

    ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query '[0].value' -o tsv)
    export ARM_ACCESS_KEY=$ACCOUNT_KEY

    
Im provider.tf wird das backend ausgetauscht

.. code-block:: yaml

    terraform {
        required_providers {
            azurerm = {
            source  = "hashicorp/azurerm"
            version = "~>3.0"
            }
        }
        backend "azurerm" {
            resource_group_name  = "<ResourceGroup-Name>"
            storage_account_name = "<storage_account_name>"
            container_name       = "tfstate"
            key                  = "terraform.tfstate"
           }
        }




