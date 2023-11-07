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
