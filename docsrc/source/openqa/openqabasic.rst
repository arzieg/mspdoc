.. _openqa_basic:

###############
openQA Allgemein
###############

DIR Structure
==============

Basis Glossar: https://open.qa/docs/#concepts

/etc/openqa   -> Konfigurationsdateien für openQA

/var/lib/openqa
    share/tests/<distribution>/ 
      * Testcode je Distribution
        * lib   - libaries
        * tests - test Module
        * schedule - Testplan
        * products - Ablageort der needles
    
    factory
      * Asstes für den Aufbau eines Tests, z.B. Image, ISO, repository ...
  
    testresults
      * logfiles

Zyklisch werden die openQA Verzeichnisse aufgeräumt. Soll etwas nicht gelöscht werden, dann liegt das in einem Unterverzeichnis "fixed". Bsp.: Ablage eines 
qemu-Images unter /var/lib/openqa/factory/hdd/fixed.
Die Aufräumaktivitäten basieren auf Jobgruppen, auf der Ebene kann man ein Regelwerk definieren, z.B. wie lange z.B. Joblogs aufbewahrt werden sollen.


Worker
=======
Die Konfigurationsparameter für die Worker werden in /etc/openqa/worker.ini gesetzt. 
Hier werden je Worker spezifische Parameter definiert:  

.. code-block:: bash
  [1]
  WORKER_CLASS = qemu_x86_64_staging,qemu_x86_64
  SUT_USER = root
  _SECRET_SUT_PASSWORD = <das root Passwort>
 
  
  [2]
  WORKER_CLASS = 64bit-ipmi, HANA1
  IPMI_HOSTNAME=<IPMI-HOSTNAME>
  IPMI_PASSWORD=<ein geheimes Passwort>
  IPMI_USER=openqa
  MAX_JOB_TIME=32000
  SUT_IP=<FQDN Hostname oder IP>
  HOSTNAME=<HOSTNAME des Workers (Public Interface)>

Settings für Worker: https://github.com/os-autoinst/os-autoinst/blob/master/doc/backend_vars.asciidoc
systemctl start openqa-worker@<nr>

GOVC vSphere API
=================
Die Anbinung an vSphere erfolgt über die vSphere API (https://github.com/vmware/govmomi/tree/main/govc)
Unter /etc/openqa gibt es 
* govc_known_hosts     KnowHost Eintrag vom vCenter
* vcenter-selfs.cert   vCenter Zertifikat
* govc.variables       Definierte Zugriffsparameter

.. code-block::bash

  export GOVC_CREDENTIALS="<Username>:<Passwort>@<vcenter-url>"

  export GOVC_URL="<vcenter-url>"
  #GOVC_USERNAME: USERNAME to use if not specified in GOVC_URL.
  export GOVC_USERNAME="<Username>"
  #GOVC_PASSWORD: PASSWORD to use if not specified in GOVC_URL.
  export GOVC_PASSWORD="<Passwort>"
  #GOVC_TLS_CA_CERTS: Override system root certificate authorities.
  #export GOVC_TLS_CA_CERTS=~/.govc_ca.crt
  # Use path separator to specify multiple files:
  #export GOVC_TLS_CA_CERTS=~/ca-certificates/bar.crt:~/ca-certificates/foo.crt
  #GOVC_TLS_KNOWN_HOSTS: File(s) for thumbprint based certificate verification.

  #Thumbprint based verification can be used in addition to or as an alternative to GOVC_TLS_CA_CERTS for self-signed certificates. Example:
  #
  # export GOVC_TLS_KNOWN_HOSTS=~/.govc_known_hosts
  # govc about.cert -u host -k -thumbprint | tee -a $GOVC_TLS_KNOWN_HOSTS
  # govc about -u user:pass@host
  # GOVC_TLS_HANDSHAKE_TIMEOUT: Limits the time spent performing the TLS handshake.

  export GOVC_INSECURE="false"
  export GOVC_TLS_KNOWN_HOSTS="/etc/openqa/govc_known_hosts"
  export GOVC_TLS_CA_CERTS="/etc/openqa/vcenter-selfs.cert"

  export no_proxy=localhost,$GOVC_URL
  export http_proxy=""
  export https_proxy=""



Einen Test schreiben
---------------------

https://open.qa/docs/#_introduction_3


1. Erstelle eine Maschine, eine Maschine hat eine Parameterisierung 
2. Erstelle ein Medium Typ (eine Distribution, eine Version, ein Flavor, eine Architektur, weitere Parameter)
3. Definiere eine Testkollektion von Tests die laufen sollen im "Test suites" Menü
4. Definiere Job Gruppen (hier laufen die Informationen aus 1-3 zusammen, diese müssen im Template angegeben werden) <https://open.qa/docs/#_configuring_job_groups_via_yaml_documents>
5. Starte Job Gruppen für die Ausführung von Tests

"Machines, mediums, test suites and job templates can all set various configuration variables. The so called job templates within the job groups define 
how the test suites, mediums and machines should be combined in various ways to produce individual 'jobs'." <https://open.qa/docs/#_introduction_3>


Perl-Modul
-----------
https://github.com/os-autoinst/os-autoinst   
/usr/lib/os-autoinst         -> Perl-Module os-autoinstallation
/usr/share/openqa/lib/OpenQA -> OpenQA Module



Testcases
----------
/var/lib/openqa/share/tests/opensuse/tests/sles4sap/sys_param_check.pm  - Einbindung von robot framework 


Memo:

/var/lib/openqa/share/tests/sle/lib/sles4sap.pm  - Bibliothek für sles4sap sapcontrol 
/var/lib/openqa/share/tests/opensuse/tests/ha    - Bibliothek für pacemaker 