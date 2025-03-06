# Cloud-Init Allgemein


## Swapfile konfigurieren

https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/create-swap-file-linux-vm#create-a-swap-partition

Bei Verwendung eines Maschinenen - Templates ist häufig die erste Disk das OS und weiterhin gibt es eine zweite Disk die nach /mnt gemountet wird. 
Hier gibt es mehrere Möglichkeiten der Swap Konfiguration

1. Create a SWAP creation script named swap.sh under /var/lib/cloud/scripts/per-boot with the following script:

```

    #!/bin/sh

    # Percent of space on the ephemeral disk to dedicate to swap. Here 30% is being used. Modify as appropriate.
    PCT=0.3

    # Location of the swap file. Modify as appropriate based on the location of the ephemeral disk.
    LOCATION=/mnt

    if [ ! -f ${LOCATION}/swapfile ]
    then

        # Get size of the ephemeral disk and multiply it by the percent of space to allocate
        size=$(/bin/df -m --output=target,avail | /usr/bin/awk -v percent="$PCT" -v pattern=${LOCATION} '$0 ~ pattern {SIZE=int($2*percent);print SIZE}')
        echo "$size MB of space allocated to swap file"

        # Create an empty file first and set correct permissions
        /bin/dd if=/dev/zero of=${LOCATION}/swapfile bs=1M count=$size
        /bin/chmod 0600 ${LOCATION}/swapfile

        # Make the file available to use as swap
        /sbin/mkswap ${LOCATION}/swapfile
    fi

    # Enable swap
    /sbin/swapon ${LOCATION}/swapfile
    /sbin/swapon -a

    # Display current swap status
    /sbin/swapon -s
```

Make the script executable:

``` 

    chmod +x /var/lib/cloud/scripts/per-boot/swap.sh

    reboot
```

2. use Cloud-Init in terraform

2.1 erzeugen einer Data Definition

```
    data "template_file" "script" {
    template = file("scripts/00-azure-cloudinit.yaml")
    }

    data "template_cloudinit_config" "config" {
    gzip          = true
    base64_encode = true

    # Main cloud-config configuration file.
    part {
        content_type = "text/cloud-config"
        content      = data.template_file.script.rendered
    }
    }
```
 
2.2 YAML File erstellen und unter scripts/<dateiname.yaml> ablegen

Die Möglichkeiten der cloud-config Parameter siehe https://cloudinit.readthedocs.io/en/latest/reference/modules.html

```

    #cloud-config
    disk_setup:
    ephemeral0:
        table_type: gpt
        layout: [66, [33, 82]]
        overwrite: True
    fs_setup:
    - device: ephemeral0.1
        filesystem: ext4
    - device: ephemeral0.2
        filesystem: swap
    mounts:
    - ["ephemeral0.1", "/mnt"]
    - ["ephemeral0.2", "none", "swap", "sw,nofail,x-systemd.requires=cloud-init.service,x-systemd.device-timeout=2", "0", "0"]
```

Selbsterklärend, bei layout sind die Zahlen wie folgt zu interpretieren:  
  66 Prozent für die erste Partition
  33 Prozent für die zweite Partition mit 82=SWAP konfiguriert.



2.3 Anpassen des Deployment-Codes für die VM, Eintrag custom_data hinzufügen mit data Ressource

```
    resource "azurerm_linux_virtual_machine" "susemanager-vm" {
    name                  = "vm-${var.vm_def.properties.name}-${var.environment}"
    ...
    custom_data           = data.template_cloudinit_config.config.rendered
```   

Ein ändern des YAML-Files führt dazu, das bei einem neuen tf apply die Maschine neu deployed werden möchte :-(
Möchte man das verhindern, dann wäre eine Möglichkeit, dass lifecycle Attribut zu setzen. 

```

      lifecycle {
        create_before_destroy = true
        ignore_changes = [
        admin_ssh_key, custom_data
        ]
    }
```

## CMD Run

https://www.linode.com/docs/guides/run-shell-commands-with-cloud-init/

here are two key features, however, that differentiate bootcmd. First, commands given in the **bootcmd** are executed early in the boot process. 
These commands run among the first tasks of system on boot. Second, they run on every system boot. 
Where **runcmd** commands **only run once, during initialization**, bootcmd commands become a part of your system’s boot process, recurring with each boot.

```
    bootcmd:
     - [ cloud-init-per, instance, example-instance-echo, echo, "Instance initialization command executed successfully!" ]
```

### Run a Bash Script

If your script is hosted and accessible remotely, the most straightforward solution is to use a wget command to download it. From there, you can use a runcmd command 
to execute the script. Object Storage can provide an effective way to host script files.
However, most use cases favor adding the shell script directly as part of the cloud-init initialization, without hosting the script file elsewhere. 
In such cases, you can use cloud-init’s write_files option to create the script file on initialization.

```
write_files:
  - path: /run/scripts/register.sh
    content: |
      #!/bin/bash
      function safe_zypper                                                                                                                                                                                             
      {                                                                                                                                                                                                                        
        local NUM_PROCS_SALT_MINION
        local LOOP
        local MAX_LOOPS=20 # wait max 5 minutes
        local LOGFILE=/var/log/cloud-init-output.log

        PROCS_ZYPPER=`ps -ef | grep [z]ypper`
        PROCS_SALT=`ps -ef | grep [s]tate.highstate`
        LOOP=1
        while [[ "$${PROCS_ZYPPER}" != "" ]] || [[ "$${PROCS_SALT}" != "" ]] && [[ $LOOP -lt $MAX_LOOPS ]]; do
                [[ "$${PROCS_ZYPPER}" != "" ]] && echo "Found zypper process $${PROCS_ZYPPER}" >> $LOGFILE
                [[ "$${PROCS_SALT}" != "" ]] && echo "Found salt highstate process $${PROCS_SALT}" >> $LOGFILE
                sleep 15
                PROCS_ZYPPER=`ps -ef | grep [z]ypper | awk '{print $2}'`
                PROCS_SALT=`ps -ef | grep [s]alt-call | awk '{print $2}'`
                let LOOP++
        done

        # kill the process
        while [[ "$${PROCS_ZYPPER}" != "" ]] || [[ "$${PROCS_SALT}" != "" ]]; do
                [[ "$${PROCS_ZYPPER}" != "" ]] && echo "Found zypper process $${PROCS_ZYPPER} and kill the process ..." >> $LOGFILE
                [[ "$${PROCS_SALT}" != "" ]] && echo "Found salt highstate process $${PROCS_SALT} and kill the process ..." >> $LOGFILE
                kill -9 $${PROCS_ZYPPER} $${PROCS_SALT}
                PROCS_ZYPPER=`ps -ef | grep [z]ypper | awk '{print $2}'`
                PROCS_SALT=`ps -ef | grep [s]alt-call | awk '{print $2}'`
        done

        [[ -n "$1" ]] && zypper $*
      }
      export HOSTNAMEFQDN="$HOSTNAME.<FQDN>"
      hostnamectl set-hostname $HOSTNAMEFQDN
      rm /etc/zypp/repos.d/*
      sleep 30
      curl -Sks https://<susemanager>/pub/bootstrap/salt-bootstrap-eddi-release-50-sle155-latest.sh | /bin/bash
      if [[ $? -ne 0 ]]
        then
          echo "Exit with error"
          exit 1
      fi
      sleep 120
      source /usr/lib/venv-salt-minion/bin/activate
      venv-salt-call state.highstate saltenv=base
      sleep 60
      # sometimes zypper has problems reading signatures in the first run, so skip-interactive
      safe_zypper -n up --skip-interactive
      # and schedule a second run
      sleep 10 
      safe_zypper -n up
    permissions: "0755"

runcmd:
  - [sh, "/run/scripts/register.sh"]
```

### Verify that Commands or Script has Run


```
    cloud-init status --wait

    sudo vi /var/log/cloud-init-output.log 
```


# Problemanalyse

## Rerun cloud-init

### Remove the logs and cache, then reboot

This method will reboot the system as if cloud-init never ran. This command does not remove all cloud-init artifacts from previous runs of cloud-init, but it will clean enough artifacts to allow cloud-init to think that it hasn’t run yet. It will then re-run after a reboot.

```
cloud-init clean --logs --reboot
```

### Run a single cloud-init module

```
cloud-init single --name cc_ssh --frequency always
```

### local

sudo mkdir -p /var/lib/cloud/seed/nocloud/
sudo cp cloud-init.yaml /var/lib/cloud/seed/nocloud/user-data
copy yaml file to /var/lib/cloud/seed/nocloud/user-data
sudo cloud-init clean --logs

-- rerun
sudo cloud-init init --local
sudo cloud-init init
sudo cloud-init modules --mode=config
sudo cloud-init modules --mode=final
-- logs
sudo tail -f /var/log/cloud-init.log
sudo tail -f /var/log/cloud-init-output.log
