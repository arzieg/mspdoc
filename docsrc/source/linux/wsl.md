# WSL 

Update WSL

`wsl --update --web-download`

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

## Some stuff

```
sudo apt install unzip
sudo apt install git
```

## Azure CLI

Install: 

```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
``` 


## Setup Oh my Posh

https://calebschoepp.com/blog/2021/how-to-setup-oh-my-posh-on-ubuntu/

### Install

```
## Install Oh my Posh
sudo wget --no-check-certificate https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
sudo chmod +x /usr/local/bin/oh-my-posh

## Download the themes
mkdir ~/.poshthemes
wget --no-check-certificate https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip -O ~/.poshthemes/themes.zip
unzip ~/.poshthemes/themes.zip -d ~/.poshthemes
chmod u+rw ~/.poshthemes/*.json
rm ~/.poshthemes/themes.zip
```

### Change prompt

add to ~/.bashrc
```
eval "$(oh-my-posh --init --shell bash --config ~/.poshthemes/atomic.omp.json)"
``` 

### Setup font

Download Font, f.i. Meslo from https://www.nerdfonts.com/font-downloads

```
cd ~
mkdir .fonts
unzip /home/c/Users/<user>/Downloads/Meslo.zip -d ~/.fonts/Meslo
fc-cache -fv
```

Im Terminal -> Einstellungen -> Profil auswählen oder neu erstellen -> unter Darstellung dann z.B. "MesloLGM NF" auswählen.

## setup tmux

Install: `sudo apt install tmux`

Config: vi ~/.tmux.conf

```
# remap prefix from 'C-b' to 'C-a'
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

# split panes using | and -
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# switch panes using Alt-arrow without prefix
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# Windows Nummerierung setzen
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
set -g history-limit 10000

# 256 Farben
set -g default-terminal "screen-256color"
set -g status-style "bg=yellow"
set -ag status-style "fg=black"
```

## setup shell

Copy .ssh/* to new place

vi ~/.bashrc
```
### start ssh-agent
if [ "${HOME}" != "/home/mobaxterm" ] ; then
        [ -f ~/.ssh/agent ] && . ~/.ssh/agent 2>&1>/dev/null
        pgrep -u $USER ssh-agent | grep -q "${SSH_AGENT_PID}"
        if [ $? -ne 0 ]
        then
          echo "Achtung, es läuft kein SSH-AGENT. Wird gestartet... "
          ssh-agent > ~/.ssh/agent
          . ~/.ssh/agent
          echo "Füge den Standard-Login-Schlüssel hinzu."
          ssh-add ~/.ssh/id_rsa
          ssh-add ~/.ssh/<mykey>.key
        fi
fi
```

## setup terraform

### Installation

```
sudo apt install gnupg
sudo apt install software-properties-common
sudo apt install curl

sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint

Check fingerprint (https://www.hashicorp.com/trust/security  Linux Package Checksum verification)
798A EC65 4E5C 1542 8C8E 42EE AA16 FCBC A621 E701


echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt-get install terraform

terraform -install-autocomplete
```

vi ~/.bashrc
```
# Terraform
alias tf="terraform"
function tfav () {
        terraform apply -auto-approve -var-file="$1"
}
```


## setup python mit venv -> noch ausarbeiten für ubuntu!

fedora: install pip
`sudo dnf install python3-pip`

## install
```
python3 -m venv project_venv
source project_venv/bin/activate
python -m pip install requests
deactivate
```

## delete
`rm -rv project_venv`

## update
`python -m pip install --update requests`

## Export / Import a WSL

https://4sysops.com/archives/export-and-import-windows-subsystem-for-linux-wsl/

Export: `wsl --export Ubuntu-24.04 c:\users\<user>\Downloads\Ubuntu2404CLAB.tar`

Import: `wsl --import Ubuntu2404CLAB c:\users\<user>\wslimages c:\users\<user>\Downloads\Ubuntu2404CLAB.tar`






## 101

### Basics

https://learn.microsoft.com/de-de/windows/wsl/basic-commands

Standardversion festlegen: `wsl --set-default-version <Version>`

Image auf eine Version ändern: `wsl --set-version Ubuntu2404CLAB 1`

WSL aktualisieren: `wsl --update --web-download`

`wsl hostname -I`: gibt die IP-Adresse Ihrer Linux-Distribution zurück, die über WSL 2 installiert wurde (die WSL 2-VM-Adresse)

`ip route show | grep -i default | awk '{ print $3}'`: Gibt die IP-Adresse des Windows-Computers zurück, wie von WSL 2 (der WSL 2-VM) dargestellt

WSL Distibution löschen: `wsl --unregister <distroName> `  https://learn.microsoft.com/de-de/windows/wsl/faq

