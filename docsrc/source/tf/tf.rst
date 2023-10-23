.. _tf_allg:

###############
Terraform
###############

Installation
=============

1. git clone https://github.com/tfutils/tfenv.git ~/.tfenv
2. PATH erweitertn
3. tfenv install latest
4. tfenv use 1.6.0
5. echo 'alias tf="terraform"' | tee -a ~/.bashrc
6. 





Prozess
========
1. terraform init (-upgrade)
2. terraform validate
3. terraform plan (-out main.tfplan)
4. terraform apply
   
  * terraform apply -var "resource_group_name=myNewResourceGroupName"   - einzelne Variablen übersteuern
  * terraform output resource_group_id  - Rückgabewerte abfrage, sofern man eine output.tf - Datei erzeugt hat

5. terraform show (was wurde angelegt)
6. terraform state list (status der angelegten Objekte)
7. terraform destroy

Remove single Resource
-----------------------
terraform state list
terraform state rm RESOURCE.ADDRESS




Variablen:
===========

.. code-block:: bash

   variable "NAME" {
    description = "..."
    type = number | list | list(number) | map(string) ...
    default = 42 | ["a","b","c"] | [1,2,3] | { key1 = "value1"
                                               key2 = "value2"
                                               key3 = "value3" }
  }

im Code wird dann via var.<VARIABLE> der Wert eingefügt


Bei terraform apply ist zu unterscheiden: 

  terraform apply   - Interaktive Angabe der Parameter 
  terraform apply -var "port=8888"   - Ohne Abfrage
  terraform apply   - Ohne Abfrage, wenn Environmentvariable definiert ist TF_VAR_port=8888

KnowHow
========
Custom-Scripts beim Anlegen von Compute Resourcen ausführen: https://sbulav.github.io/terraform/terraform-azurerm-compute-custom-data/



AZTFExport
===========

aztfexport ist ein Tool, um eine vorhanden Definition einer Azure Infrastruktur in ein Terraform Script zu übertragen. 

Quelle: https://learn.microsoft.com/de-de/azure/developer/terraform/azure-export-for-terraform/export-terraform-overview

Source: https://github.com/Azure/aztfexport/releases

Installation
-------------
git clone https://github.com/Azure/aztfexport.git oder

Ubuntu
.......
curl -sSL https://packages.microsoft.com/keys/microsoft.asc > /etc/apt/trusted.gpg.d/microsoft.asc
ver=20.04 # or 22.04
apt-add-repository https://packages.microsoft.com/ubuntu/${ver}/prod
apt-get install aztfexport


Befehle
-------
aztfexport config set telemetry_enabled false   -> telemetry Daten nicht senden

aztfexport query -n "resourceGroup =~ 'myResourceGroup' and type contains 'Microsoft.Network'"  -> Query Beispiel

aztfexport resource-group --non-interactive --hcl-only myResourceGroup  -> Export Beispiel
