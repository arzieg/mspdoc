.. _cloud-init_allg:

########################
Cloud-Init Allgemein
########################


Swapfile konfigurieren
=======================
https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/create-swap-file-linux-vm#create-a-swap-partition

Bei Verwendung eines Maschinenen - Templates ist häufig die erste Disk das OS und weiterhin gibt es eine zweite Disk die nach /mnt gemountet wird. 
Hier gibt es mehrere Möglichkeiten der Swap Konfiguration

1. Create a SWAP creation script named swap.sh under /var/lib/cloud/scripts/per-boot with the following script:

.. code-block:: shell

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

Make the script executable:

.. code-block:: 

    chmod +x /var/lib/cloud/scripts/per-boot/swap.sh

reboot

2. use Cloud-Init in terraform

2.1 erzeugen einer Data Definition

.. code-block:: shell

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

 
2.2 YAML File erstellen und unter scripts/<dateiname.yaml> ablegen

Die Möglichkeiten der cloud-config Parameter siehe https://cloudinit.readthedocs.io/en/latest/reference/modules.html

.. code-block:: shell

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


Selbsterklärend, bei layout sind die Zahlen wie folgt zu interpretieren:  
  66 Prozent für die erste Partition
  33 Prozent für die zweite Partition mit 82=SWAP konfiguriert.



2.3 Anpassen des Deployment-Codes für die VM, Eintrag custom_data hinzufügen mit data Ressource

.. code-block:: 

    resource "azurerm_linux_virtual_machine" "susemanager-vm" {
    name                  = "vm-${var.vm_def.properties.name}-${var.environment}"
    ...
    custom_data           = data.template_cloudinit_config.config.rendered
      

Ein ändern des YAML-Files führt dazu, das bei einem neuen tf apply die Maschine neu deployed werden möchte :-(
Möchte man das verhindern, dann wäre eine Möglichkeit, dass lifecycle Attribut zu setzen. 

.. code-block:: shell

      lifecycle {
        create_before_destroy = true
        ignore_changes = [
        admin_ssh_key, custom_data
        ]
    }