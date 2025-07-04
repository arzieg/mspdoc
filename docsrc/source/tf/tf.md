# Terraform

## Installation


1. git clone https://github.com/tfutils/tfenv.git ~/.tfenv
2. PATH erweitertn
3. tfenv install latest
4. tfenv use 1.6.0
5. echo 'alias tf="terraform"' | tee -a ~/.bashrc
 


## Version

Providerversionen in der .terrafrom.lock.hcl


## Prozess

1. terraform init (-upgrade)
2. terraform validate
3. terraform plan (-out main.tfplan)
4. terraform apply (-auto-approve)
   
  * terraform apply -var "resource_group_name=myNewResourceGroupName"   - einzelne Variablen übersteuern
  * terraform output resource_group_id  - Rückgabewerte abfrage, sofern man eine output.tf - Datei erzeugt hat
  * terraform apply -refresh-only       - Status in TF aktualisieren, wenn man z.B. manuell eine Ressource gelöscht hat. 

5. terraform show (was wurde angelegt)
6. terraform state list (status der angelegten Objekte)
7. terraform destroy

### Remove single Resource ohne sie in azure zu löschen
---------------------------------------------------------
terraform state list
terraform state rm RESOURCE.ADDRESS

### Resource wurde in der Cloud angelegt, wird in TF nachgebaut:
---------------------------------------------------------------
tf plan -refresh-only   (um den Drift zw. OnPrem und Cloudconfig festzustellen)
tf import azurerm_network_security_group.<SG-NAME> <ID>
tf plan -refresh-only   (sollte dann idealerweise identisch sein)


https://itnext.io/terraform-using-import-and-some-hidden-pitfalls-f28f12415836


### terraform workspace:

https://spacelift.io/blog/terraform-workspaces

```
terraform workspace --help
Usage: terraform [global options] workspace

  new, list, show, select, and delete Terraform workspaces.

Subcommands:
    delete    Delete a workspace
    list      List Workspaces
    new       Create a new workspace
    select    Select a workspace
    show      Show the name of the current workspace
```

Create: 

```
terraform workspace new test_workspace
Created and switched to workspace "test_workspace"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
``` 
When we create a new workspace, Terraform creates a corresponding new state file in the same remote backend that is configured initially. It is important to note that the backend being used should also be able to support the workspaces.




## Debug


1. Set Log level using TF_LOG (export TF_LOG="DEBUG")
2. Set up log file using TF_LOG_PATH (export TF_LOG_PATH="/home/vagrant/terraform-ec2-aws/terraform-debug.log")

### terraform console
------------------
ermöglicht den schnellen Blick auf Variablen und wie diese in tf umgesetzt werden

Beispiel: 

```
terraform console

var.hostname    -> zeige die Variable Hostname
local.var1     -> zeige die locale Variable var1
split(",", "follow,this,blog")  -> was erzeugt die Funktion split für ein Output
```


### Variablen:


```

  variable "NAME" {
  description = "..."
  type = number | list | list(number) | map(string) ...
  default = 42 | ["a","b","c"] | [1,2,3] | { key1 = "value1"
                                              key2 = "value2"
                                              key3 = "value3" }
}
```

im Code wird dann via var.\<VARIABLE\> der Wert eingefügt


Bei terraform apply ist zu unterscheiden: 

  terraform apply   - Interaktive Angabe der Parameter 
  terraform apply -var "port=8888"   - Ohne Abfrage
  terraform apply   - Ohne Abfrage, wenn Environmentvariable definiert ist TF_VAR_port=8888

## KnowHow

Custom-Scripts beim Anlegen von Compute Resourcen ausführen: https://sbulav.github.io/terraform/terraform-azurerm-compute-custom-data/



## AZTFExport


aztfexport ist ein Tool, um eine vorhanden Definition einer Azure Infrastruktur in ein Terraform Script zu übertragen. 

Quelle: https://learn.microsoft.com/de-de/azure/developer/terraform/azure-export-for-terraform/export-terraform-overview

Source: https://github.com/Azure/aztfexport/releases

### Installation
-------------
git clone https://github.com/Azure/aztfexport.git oder

Ubuntu
.......
curl -sSL https://packages.microsoft.com/keys/microsoft.asc > /etc/apt/trusted.gpg.d/microsoft.asc
ver=20.04 # or 22.04
apt-add-repository https://packages.microsoft.com/ubuntu/${ver}/prod
apt-get install aztfexport

unter 24.04

go install github.com/Azure/aztfexport@latest
binary liegt im $HOME/go/bin/aztfexport


### Befehle
-------
aztfexport config set telemetry_enabled false   -> telemetry Daten nicht senden

aztfexport query -n "resourceGroup =~ 'myResourceGroup' and type contains 'Microsoft.Network'"  -> Query Beispiel

aztfexport resource-group --non-interactive --hcl-only myResourceGroup  -> Export Beispiel

aztfexport resource --non-interactive --hcl-only \<RessourceID, also /subscriptions/...\>  -> einzelne Ressource exportieren

  Szenario: es wurde etwas manuell hinzugefügt. Mittels aztfexport das exportieren und dann in den Gesamtexport integrieren
    aztfexport resource -o tempdir --hcl-only \<resource_id\>  (-o exportiere in das Verzeichnis)
    aztfexport map --append `./tempdir/aztfexportResourceMapping.json` (füge das exportierte in den Gesamtexport ein)
    terraform init --upgrade
    terraform plan  (sollte dann keine Abweichungen anzeigen)

#### Suchen nach einer speziellen Ressource
az account list --output table
az account set --subscription="\<ID\>"
az keyvault show --name <ressource> --query id --output tsv
  -> ressourcen-id kommt raus
aztfexport resource <ressourcen-id>





az account show --output table\
az account list --output table\
az account set --subscription="\<ID\>"