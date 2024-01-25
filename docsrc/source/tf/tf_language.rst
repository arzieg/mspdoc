.. _tf_language:

##############################
Terraform Language
##############################

Loops 
=====

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

Einschränkungen: 

* A given resource or module block cannot use both count and for_each.
* for_each keys cannot be the result (or rely on the result of) of impure functions, as their evaluation is deferred during the main evaluation step.


for
----

Die for-Schleife bezieht sich hier auf einzelne Werte und nicht auf Ressourcen oder Inline Blocks wie bei counter oder for_each.

Allgemeine Syntax:

``[FOR <ITEM> in <LIST> : <OUTPUT>]``

Loop über list und output map

``{for <ITEM> in <LIST> : <OUTPUT_KEY> => <OUTPUT_VALUE>}``

Loop über map und output map

``{for <KEY>, <VALUE> in <MAP> : <OUTPUT_KEY> => <OUTPUT_VALUE>}``


Beispiele Uppercase:

.. code-block:: shell

  variable "hosts" {
    description = "Hostnames"
    type = list(string)
    default = ["asterix", "obelix", "idefix"]
    }

  output "upper_hosts" {
    value = [for hosts in var.hosts : upper(hosts)]
    }

  
Beispiel map:

.. code-block:: shell

    variable "cf_interface" {
      description = "Cache Fusion Interface"
      type        = map(string)
      default = {
        "asterix" = "192.168.0.1"
        "obelix"  = "192.168.0.3"
        "idefix"  = "192.168.0.5"
      }
    }

    output "host_to_cf" {
      value = { for host, cf in var.cf_interface : host => cf }
    }


Loops with string directive
----------------------------

Terraform unterstützt zwei Formen der string derictive: in for-loops und conditionals

Syntax:

``%{ for <ITEM> in <COLLECTION> } <BODY> %{ endfor }``

COLLECTION ist dabei eine LIST oder MAP worüber iteriert wird. ITEM ist die lokale Variable die den Wert erhält.

Die Syntax ist ähnlich zu Jinja2


Beispiel for: 

.. code-block:: shell

  variable "hosts" {
  description = "Hosts"
  type        = list(string)
  default     = ["asterix", "obelix", "idefix"]
  }

  output "for_directive" {
    value = <<EOF
      %{~for hosts in var.hosts}     <-- die ~ eleminiert die Leerzeichen und /n
        ${hosts}                     <-- hier wird Variable abgefragt, daher $
      %{~endfor}                     <-- muss auch hier stehen, sonst geht es nicht
      EOF
  }

Beispiel if:

.. code-block:: shell



IF (conditionals)
===================

Syntax (wie c)

``<CONDITION> ? <TRUE VAL> : <FALSE VAR>``

Beispiel mit einer for_each Schleife, wo man das per Host setzen kann:

.. code-block:: shell
 
  variable "autoshutdown_on_host" {
    type = list(string)
  }
  
  resource "azurerm_dev_test_global_vm_shutdown_schedule" "nginxVMshutdown" {
    for_each              = toset(var.hostname)
    enabled               = contains(var.autoshutdown_on_host, each.key) ? true : false
    daily_recurrence_time = "1800"
    location              = var.location
    timezone              = "W. Europe Standard Time"
    virtual_machine_id    = azurerm_linux_virtual_machine.nginx[each.key].id
    notification_settings {
      enabled = false
    }
  }


Ein if - else Konstrukt mit count ist schwieriger, da man hier mit einfachen, sich gegenseitig ausschließenden IF Anweisungen arbeiten muss. 

Beispiel Verwendung eines anderen Scriptes als POST Aktivität beim Erzeugen einer VM:

.. code-block:: shell

  variable "enable_new_script" {
    description = "Verwende neues Script?"
    type = bool
    }

  # dann ein Data mit zwei sich gegenseitig ausschließenden IF Statements 
  data "template_file" "host_script_old" {
    count = var.enable_new_script ? 0 : 1
    template = file("${path.module}/host_script.sh")
  }

  data "template_file" "host_script_new" {
    count = var.enable_new_script ? 1 : 0    <- hier umgekehrte Logik um das else hinzubekommen
    template = file("${path.module}/host_script_new.sh")
  }

  # dann in der Erzeugung der VM
  ...
  user_data = (
    length(data.template_file.host_script[*]) > 0       <- nur eines ist definiert durch if-else oben
      ? data.template_file.host_script_old[0].rendered  <- der Wert steht dann an Index-Stelle 0
      : data.template_file.host_script_new[0].rendered
  )


Conditionals mit for_each
--------------------------

.. code-block:: shell

  var "autoshutdown" {
    description = "Enable autoshutdown"
    type = bool
  }

  resource "azurerm_dev_test_global_vm_shutdown_schedule" "nginxVMshutdown" {
    for_each              = var.autoshutdown
    for_each              = toset(var.hostname)
    daily_recurrence_time = "1800"
    location              = var.location
    timezone              = "W. Europe Standard Time"
    virtual_machine_id    = azurerm_linux_virtual_machine.nginx[each.key].id
    notification_settings {
      enabled = false
    }
}



Tags
=====

Tags können als Key-Value Pairs angegeben werden. 

custom_tags      = { "Umgebung" = "Testumgebung", "Owner" = "Max Mustermann", "PSP" = "IT.001-100" }

Im tfstate dann z.B. direkt angeben über

tags = custom_tags 


Datenstrukturen
================

maps
-----
https://spacelift.io/blog/terraform-map-variable
https://spacelift.io/blog/terraform-flatten

Einfache als auch verschachtelte Key-Value Paar können über maps erstellt werden. 

.. code-block:: shell

  ## Hostname, static IP, Autoshutdown definition
  variable "vm_name" {
    type = map(object({
      ip           = string
      autoshutdown = bool

    }))
    default = {
      "lascismp1" = {
        ip           = "10.100.1.5",
        autoshutdown = true
      }
      "lascismp2" = {
        ip           = "10.100.1.6",
        autoshutdown = false
      }
    }
  }

Dieser kann auch in einer for_each Schleife verwendet werden. 

.. code-block:: shell

  resource "azurerm_dev_test_global_vm_shutdown_schedule" "sumaVMshutdown" {
    for_each              = var.vm_name
    enabled               = each.value.autoshutdown
    daily_recurrence_time = "1800"
    location              = var.location
    timezone              = "W. Europe Standard Time"
    virtual_machine_id    = azurerm_linux_virtual_machine.susemanager-vm[each.key].id
    notification_settings {
      enabled = false
    }
  }







Good to Know
==============

Die Reihenfolge des Löschens und Anlegens von Ressourcen kann man ggfs. beeinflussen, um ein Zero-Downtime Deployment hinzubekommen. Mit der lifecycle Option in 
vielen Ressourcen kann man das beeinflussen: 

Beispiel: ignoriere etwas

.. code-block:: shell

   lifecycle {
    ignore_changes = [
      custom_data,
    ]
  }


Beispiel: erzeuge erst etwas und zerstörre danach

.. code-block:: shell

   lifecycle {
     create_before_destroy = true
  }


