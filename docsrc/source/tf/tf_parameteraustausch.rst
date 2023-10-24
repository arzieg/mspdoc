.. _tf_parameteraustausch:

##############################
Terraform Parameteraustausch
##############################

Wenn man sich eine Verzeichnisstruktur aufgebaut hat, um die Umgebungen voneinander zu kapseln, 
muss man ggfs. auf Werte zugreifen, die in einer anderen Verzeichnisstruktur erstellt wurden. 
Da jedes gekapselte Verzeichnis sein eigenes tfstate hat, kann man nicht so einfach auf die Werte 
zugreifen. 
In dem tfstate kann man über Output-Variablen auf Werte zugreifen, die man im weiteren Code an anderer
Stelle verwenden möchte. 

Beispiel: Erstellung der Netzwerkstruktur in einem eigenen Zweig (also mit eigenem tfstate)
 
1. Pfad des Storageproviders

    .. code-block:: yaml

        ...
        key                  = "mgmt/vpc/terraform.tfstate"
        ...

2. Erstellen der Output-Variablen

   Erstellen einer outputs.tf Datei mit den Werten, auf die andere Terraform - Scripte zugreifen müssen. Bsp.:

   .. code-block:: yaml

        ...
        output "AzureBastionSubnetID" {
        value = azurerm_subnet.AzureBastionSubnet.id
        }
        ...

   In diesem Beispiel wird Wert von azurerm_subnet.AzureBastionSubnet.id der frei wählbaren Variable AzureBastionSubnetID zugeordnet. 
   Im tfstate-File werden diese key-value pairs abgespeichert.
 
3. Einlesen der Output-Variablen

   Terraform kann an anderer Stelle auf die Werte Read-Only zugreifen. Bsp.: in einem anderen gekapselten Bereich wird der AzureBastionHost erstellt. 
   Um nun auf die oben definierte SubnetID zugreifen zu können ist folgender Code notwendig:


    .. code-block:: yaml

        data "terraform_remote_state" "mgmt-vpc" {
        backend = "azurerm"
        config = {
            resource_group_name  = "<Resourcengruppe>"
            storage_account_name = "<Storageaccount>"
            container_name       = "tfstate"
            key                  = "mgmt/vpc/terraform.tfstate"
            }
        }
        ...
    
        resource "azurerm_bastion_host" "bastionVM" {
        location            = var.location
        name                = "<Name>"
        resource_group_name = var.rg
        sku                 = "Standard"
        tunneling_enabled   = true
        ip_configuration {
            name                 = "IpConf"
            public_ip_address_id = azurerm_public_ip.stage-vnet-bastion-public-ip.id
            subnet_id            = data.terraform_remote_state.mgmt-vpc.outputs.AzureBastionSubnetID

          }
        }



