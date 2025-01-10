
git clone https://github.com/oracle/docker-images.git

cd /home/arne/dev/docker-images/OracleDatabase/SingleInstance/dockerfiles/23.6.0


[oracle@localhost dockerfiles]$ ./buildContainerImage.sh -h

./buildContainerImage.sh -v 23.6.0 -t ora236 -f

mkdir /var/tmp/oradata
chmod 777 /var/tmp/oradata



podman run --name ora23 \
-p 1521:1521 \
-e ORACLE_PWD=start123 \
-e ORACLE_CHARACTERSET=AL32UTF8 \
-e ENABLE_ARCHIVELOG=false \
-e ENABLE_FORCE_LOGGING=false \
-v /var/tmp/oradata:/opt/oracle/oradata \
localhost/ora236

Parameters:
   --name:        The name of the container (default: auto generated)
   -p:            The port mapping of the host port to the container port.
                  Only one port is exposed: 1521 (Oracle Listener)
   -e ORACLE_PWD: The Oracle Database SYS, SYSTEM and PDBADMIN password (default: auto generated)
   -e ORACLE_CHARACTERSET:
                  The character set to use when creating the database (default: AL32UTF8)
   -e ENABLE_ARCHIVELOG: 
                  To enable archive log mode when creating the database (default: false)
   -e ENABLE_FORCE_LOGGING:
                  To enable force logging mode when creating the database (default: false)
   -v /opt/oracle/oradata
                  The data volume to use for the database.
                  Has to be writable by the Unix "oracle" (uid: 54321) user inside the container.
                  If omitted the database will not be persisted over container recreation.
   -v /opt/oracle/scripts/startup 
                  Optional: A volume with custom scripts to be run after database startup.
                  For further details see the "Running scripts after setup and on startup" section below.
   -v /opt/oracle/scripts/setup 
                  Optional: A volume with custom scripts to be run after database setup.
                  For further details see the "Running scripts after setup and on startup" section below.


Passwort ändert
p exec ora23 /opt/oracle/setPassword.sh start123

