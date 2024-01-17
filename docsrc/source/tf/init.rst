.. _tf_init:

########################
Project Initialization
########################


Leere Subscription
====================
1. ResourceGroup anlegen
   1.1 Per PIM die Rechte holen
       https://portal.azure.com/#view/Microsoft_Azure_PIMCommon/ActivationMenuBlade/~/azurerbac
       Hier zu der Subscription sich Rechte einholen (Contributor)
   1.2 Dann die ResourceGroup erstellen unter der Subscription
2. Terraform Service Principal anlegen
   2.1 Per PIM die Rechte als Application Developer holen (link wie oben nur Menuepunkt "Microsoft Entra Roles")
       https://portal.azure.com/?feature.msaljs=true#view/Microsoft_Azure_PIMCommon/ActivationMenuBlade/~/aadmigratedroles/provider/aadroles
   2.2 User anlegen 
       az ad sp create-for-rbac --name <service principal name> --role Contributor --scopes /subscriptions/<Subscription>


