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

    
Im provider.tf wird nun das Backend ausgetauscht. 

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
            key                  = "global/storage/terraform.tfstate"
           }
        }

Danach noch einmal einen *tf.init*. Man wird dann gefragt, das Statefile in das neue Backend zu schreiben.

.. code-block:: 

   Initializing the backend...
   Do you want to copy existing state to the new backend?
   Pre-existing state was found while migrating the previous "local" backend to the
   newly configured "azurerm" backend. No existing state was found in the newly
   configured "azurerm" backend. Do you want to copy this state to the new "azurerm"
   backend? Enter "yes" to copy and "no" to start with an empty state.
   
3. Aufbau der weiteren Verzeichnisstruktur
  
   Wenn man eine weitere Verzeichnisstruktur aufbaut (z.B. eine Unterscheidung in dev/test/prod), dann sollte auch zu jeder Umgebung ein eigenes Terraform - Statusfile existieren.
   In dem jeweiligen Verzeichnis gibt man in der provider.tf dann den Key an, wo der Status abgespreichert werden soll.
   Bsp.:

   .. code-block:: yaml

    ...
    backend "azurerm" {
        resource_group_name  = "<ResourceGroup-Name>"
        storage_account_name = "<storage_account_name>"
        container_name       = "tfstate"
        key                  = "dev/terraform.tfstate"   
    }





