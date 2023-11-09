.. _tf_production:

########################
Terraform in Produktion
########################

Checklist
=============
.. csv-table:: Infrastructure Checklist
   :widths: 20, 40, 40
   :header rows: 1

  Aufgabe,Beschreibung,Tools
  Installation,Installation aller Komponenten,Ansible Salt bash chef puppet
  Konfiguration,Konfiguration der Software  Ports  Zertifikate,Ansible Salt bash chef puppet
  Provisionierung,Bereitstellung der Infrastruktur Server Berechtigungen Load Balancer …,Terraform Cloudformation
  Deploy,Ausrollen der Infrastruktur Life Cycle Management Zero Downtime blue green tests  …,Terraform Cloudformation Kubernetes ECS
  Hochverfügbarkeit,HA in Prozessen  Servern  Services  DC und Regionen,mehrere DC  multi Region  Replication  Autoscaling  load balancing
  Skalierung,Scale Out/Down horizontal/vertikal,auto scaling  Replikation  Caching  Hybrid  Teilung
  Performance,Optimierung CPU  RAM  Storage  Netzwerk  benchmark  load test  profiling,div. Tools
  Netzwerk,statische/dynamische IP  Routen  Firewalls  DNS  ssh  vpn  …,VPC  Firewall  Router  DNS  VPN
  Sicherheit,Verschlüsselung  Authentifizierung  Authensierung  Umgang mit Secrets  Härtung,CIS  KMS  Zertifikate  …
  Metriken,technische  business Metriken  apps  server  events  tracing  Alarmierung,DataDog  Honeycomb
  Logs,log   rotate log  log  Analyse  zentrale Logablage,ELK  Cloud Watch
  Backup und Restore,backup  snap  restore Verfahren,Relication  Networker
  Kostenoptimierung,auto scaling  Verwendung ungenutzter Ressourcen  Spot Ressourcen,Auto Skalierung  Spot Instanzen  Reservierte Instanzen
  Dokumentation,Dokumentation von Architektur  Code  Betrieb  Playbooks  …,wiki  confluence
  Tests,Automatische Test (Code  Integration  Desaster),Terratest  inspec  serverspec  kitchen terraform


Module-Directory
=================

Ziel ist es möglichst viele kleine Module zu erstellen die einzelne Aufgaben erledigen (wie Funktionen)

Verzeichnis-Bsp:

.. code-block:: shell

   modules
   examples (zeigt wie man mit dem Modul umzugehen hat)
      Aufbau loesungsbaustein 1
      Aufbau loesungsbaustein 2
         eine Instanz
         Autoscaling
         load-balancer
         ...
      Aufbau loesungsbaustein 3
      ...
   modules
      Template loesungsbaustein 1
      Template loesungsbaustein 2
      ...
   test
      Aufbau loesungsbaustein 1
      Aufbau loesungsbaustein 2
      ...

TF Version-Pinning
====================

Jedes Modul sollte mit einer Versionsprüfung der Terraform API definiert werden

.. code-block:: shell

   terraform {
    required_version =">=0.12, < 0.13"  # eher softe Versionsprüfung
    }

   terraform {
     required_version = "=0.12.0" #striktere Versionsprüfung, da genaue Version verlangt
   }

Provider Pinnen
================
Provider sollten ebenfalls einer Versionsprüfung unterzugen werden:

.. code-block:: shell

   terraform {
      required_providers {
         azurerm = {
            source  = "hashicorp/azurerm"
            version = "~>3.0"
         }
      }
   }

Module Versionieren
====================
Module sind in github abzulegen und zu versionieren per tags

.. code-block:: shell

   module "test" {
     source = "git@github.com:/foo/modules.git//services/test?ref=v0.0.3"
     ...
   }

Nichts das Rad neu erfinden:
============================
Quelle der Inspiration https://registry.terraform.io/


Provisioniers
==============

TF provisioniers können als **"letztes"** Mittel verwendet werden, um Dinge umzusetzen. Letztendlich sind es Scripte die auf lokalen oder entfernten Maschinen abgestezt werden können.
(https://developer.hashicorp.com/terraform/language/resources/provisioners/syntax)

Unterschieden wird zwischen: 
* local-exec  = lokale Exekutoren
* remote-exec = remote Exekutoren
* file        = file copy (per ssh oder winrm)

.. code-block:: shell

   resource "local_file" "foo" {
      content  = "foo!"
      filename = "${path.module}/foo.bar"

      provisioner "local-exec" {        <- baut man dann in den Block mit ein
         command = "echo \"Hello World from $(uname)\""
         }
   }


External Data source
======================

Ziel: Script ausführen und Rückgabe erhalten. Dies auch nur als **"letztes"** Mittel verwenden, wenn das Modul, die Ressource nicht die notwendigen Informationen liefert
Hierbei wird JSON geschrieben / ausgelesen. Beispiel: lesen aus stdin und Ausgabe an stdout. Es muss ein json Objekt ausgegeben werden, hier bietet sich jq an. 

.. code-block:: shell

   data "external" "echo" {
      program = ["bash", "-c", "cat /dev/stdin"]
      query = {
         "name"  = "apple"
         "color" = "green"
         "price" = "1.2"
      }
    }

   output "echo" {
      value = data.external.echo.result
   }




