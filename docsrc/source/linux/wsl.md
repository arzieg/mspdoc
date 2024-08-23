# WSL 

https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/guides/install-ubuntu-wsl2/

List avaiable Distros: `wsl --list --online`

Install: wsl --install -d Ubuntu-24.04 --web-download  (ohne Webdownload will wsl das aus dem MS Store runterladen, was nicht zugelassen ist)

Update: 
```
sudo apt update
sudo apt full-upgrade -y
```

Set WSL Version 2: 
```
wsl --list --verbose
wsl --set-version <distribution name> 2
``` 


## Setup Oh my Posh

https://calebschoepp.com/blog/2021/how-to-setup-oh-my-posh-on-ubuntu/



## 101

https://learn.microsoft.com/de-de/windows/wsl/basic-commands

Standardversion festlegen: `wsl --set-default-version <Version>`

WSL aktualisieren: `wsl --update --web-download`

`wsl hostname -I`: gibt die IP-Adresse Ihrer Linux-Distribution zurück, die über WSL 2 installiert wurde (die WSL 2-VM-Adresse)

`ip route show | grep -i default | awk '{ print $3}'`: Gibt die IP-Adresse des Windows-Computers zurück, wie von WSL 2 (der WSL 2-VM) dargestellt


