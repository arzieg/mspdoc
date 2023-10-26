.. _tf_language:

##############################
Terraform Language
##############################

Loops
======

* *count* parameter, loop over resources
* *for_each* expression, to loop over resources and inline blocks within a resource
* *for* expression. loop over lists and maps
* *string* directive. loop over list and maps within a string

count
-----

.. code-block:: shell

    variable "hostnames" {
      description = "Hostnames to create"
      type = list(string)
      default = ["asterix", "obelix", "majestetix"]
      }

    resource "azurerm_linux_virtual_machine" "nginx" {
      admin_username        = "admin"
      eviction_policy       = "Deallocate"
      location              = var.location
      count                 = length(var.hostname)   <- länge des Arrays
      name                  = var.hostname[count.index]  <- array ansprechen
      ...

count kann bei Resourcen verwendet werden. Beim Output funktioniert dies nicht.

Nachteile: 

* *count* kann nicht im "Inline"-Block verwendet werden. 
* wenn man eine Änderung an den Indizies vornimmt, dann gibt es ungewollte Verschiebungen. Beispiel Host-1 soll gelöscht werden, dann kann man das aus dem Index entfernen. Dann möchte TF aber Host-2 in Host-1 umbenennen usw., das ist nicht das was man möchte. 
* Vielleicht gibt es einen Weg, aber beim Output darf man kein count.index angeben. Ergo, konnte ich nicht die einzelnen Werte ausgeben.


for_each
---------
Schleife über lists, sets und maps. 

.. code-block:: shell

    variable "hostname" {
      description = "Hostnames to create"
      type = list(string)
      default = ["asterix", "obelix", "majestetix"]
      }

    resource "azurerm_linux_virtual_machine" "nginx" {
      admin_username        = "admin"
      eviction_policy       = "Deallocate"
      location              = var.location
      for_each              = toset(var.hostname) 
      name                  = each.value
      ...

toset konvertiert var.hostnames, da for_each nur sets und maps in einer Ressource unterstützt.
Beim Output ist die Notation wie folgt: 

.. code-block:: shell

    output "nginx_private_ip" {
      value = values(azurerm_linux_virtual_machine.nginx)[*].private_ip_address
    }


Sensible Daten:

For example, if you would like to call keys(local.map), where local.map is an object with sensitive values (but non-sensitive keys), you can create a value to pass to for_each with toset([for k,v in local.map : k]).


Referring to Instances:

When for_each is set, Terraform distinguishes between the block itself and the multiple resource or module instances associated with it. Instances are identified by a map key (or set member) from the value provided to for_each.

* <TYPE>.<NAME> or module.<NAME> (for example, azurerm_resource_group.rg) refers to the block.
* <TYPE>.<NAME>[<KEY>] or module.<NAME>[<KEY>] (for example, azurerm_resource_group.rg["a_group"], azurerm_resource_group.rg["another_group"], etc.) refers to individual instances.

This is different from resources and modules without count or for_each, which can be referenced without an index or key.


Tags
=====

Tags können als Key-Value Pairs angegeben werden. 

custom_tags      = { "Umgebung" = "Testumgebung", "Owner" = "Max Mustermann", "PSP" = "IT.001-100" }

Im tfstate dann z.B. direkt angeben über

tags = custom_tags 
