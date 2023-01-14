.. _msf:

##################
MSF Basisbefehle
##################

Start von Metasploit über die msfconsole

.. code-block:: bash

    msfconsole

Wenn ein "offener" Service gefunden wurde, würde man in MSF nach einem exploit oder später ein payload suchen. 
Beispiel postgres.

.. code-block:: bash

    search postgres

       #   Name                                                        Disclosure Date  Rank       Check  Description
   -   ----                                                        ---------------  ----       -----  -----------
   0   auxiliary/server/capture/postgresql                                          normal     No     Authentication Capture: PostgreSQL
   1   post/linux/gather/enum_users_history                                         normal     No     Linux Gather User History
   2   exploit/multi/http/manage_engine_dc_pmp_sqli                2014-06-08       excellent  Yes    ManageEngine Desktop Central / Password Manager LinkViewFetchServlet.dat SQL Injection
   3   exploit/windows/misc/manageengine_eventlog_analyzer_rce     2015-07-11       manual     Yes    ManageEngine EventLog Analyzer Remote Code Execution
   4   auxiliary/admin/http/manageengine_pmp_privesc               2014-11-08       normal     Yes    ManageEngine Password Manager SQLAdvancedALSearchResult.cc Pro SQL Injection
   5   auxiliary/analyze/crack_databases                                            normal     No     Password Cracker: Databases
   6   exploit/multi/postgres/postgres_copy_from_program_cmd_exec  2019-03-20       excellent  Yes    PostgreSQL COPY FROM PROGRAM Command Execution
   7   exploit/multi/postgres/postgres_createlang                  2016-01-01       good       Yes    PostgreSQL CREATE LANGUAGE Execution
   8   auxiliary/scanner/postgres/postgres_dbname_flag_injection                    normal     No     PostgreSQL Database Name Command Line Flag Injection
   9   auxiliary/scanner/postgres/postgres_login                                    normal     No     PostgreSQL Login Utility
   10  auxiliary/admin/postgres/postgres_readfile                                   normal     No     PostgreSQL Server Generic Query
   11  auxiliary/admin/postgres/postgres_sql                                        normal     No     PostgreSQL Server Generic Query
   12  auxiliary/scanner/postgres/postgres_version                                  normal     No     PostgreSQL Version Probe
   13  exploit/linux/postgres/postgres_payload                     2007-06-05       excellent  Yes    PostgreSQL for Linux Payload Execution
   14  exploit/windows/postgres/postgres_payload                   2009-04-10       excellent  Yes    PostgreSQL for Microsoft Windows Payload Execution
   15  auxiliary/scanner/postgres/postgres_hashdump                                 normal     No     Postgres Password Hashdump
   16  auxiliary/scanner/postgres/postgres_schemadump                               normal     No     Postgres Schema Dump
   17  auxiliary/admin/http/rails_devise_pass_reset                2013-01-28       normal     No     Ruby on Rails Devise Authentication Password Reset


* Der Name gibt den Ort des Ruby-Scripts an.
* Rank gibt die an, wie gut ein exploit ist. 
* Check gibt an, ob vorher geprüft werden kann, ob ein Exploit auf dem Zielsyste läuft oder nicht.
* Disclosure Date gibt an, seit wann der Exploit bekannt ist.

Mit use wird ein Modul ausgeählt. Entweder als Name angeben inkl. vollständigen Pfad oder aber die Nummer.

.. code-block:: Shell

    use auxiliary/scanner/postgres/postgres_login
    oder
    use 9

Mit info erhält man Informationen zu dem Module. Mit options werden die definierbaren Parameter angezeigt. 
Mit set wird ein Parameterwert gesetzt

.. code-block:: Shell

    set RHOST <Ziel IP>
    show options

Mit check kann geprüft werden, ob der exploit funktioniert oder nicht sofern das Modul das unterstützt.

Der Start erfolgt dann mit "run". 

.. code-block:: Shell

    ...
    [-] <IP> - LOGIN FAILED: admin:password@template1 (Incorrect: FATAL      VFATAL  C28P01  Mpassword authentication failed for user "admin"Fauth.c L330    Rauth_failed)
    [+] <IP> - Login Successful: admin:admin@template1   <-- Treffer
    [-] <IP> - LOGIN FAILED: postgres:postgres@template1 (Incorrect: FATAL   VFATAL  C28P01  Mpassword authentication failed for user "postgres"     Fauth.c L330    Rauth_failed)






