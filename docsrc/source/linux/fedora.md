# Fedora

## dnf

Der config-manager muss als plugin nachinstalliert werden. 

dnf install 'dnf5-command(config-manager)'


### find

dnf search PAKET

dnf provides PAKET

dnf download --url PAKET 

dnf group list 

### info

dnf info PAKET

dnf info @GRUPPE

dnf repoquery --list PAKET

### install and remove

dnf install PAKET

dnf install @GRUPPE

dnf remove PAKET

### Download

dnf download PAKET

dnf download --resolve PAKET  - incl. missing packages, no install

dnf download --resolve --alldeps PAKET 

dnf download --source PAKET   - download source

### Dependencies

dnf repoquery --deplist PAKET

dnf --allowerasing

dnf --skip-broken 

dnf builddep PAKET

### Repos

dnf repolist --all   - show all repos

dnf repolist         - show enabled repolist

dnf config-manager --enable REPO

dnf config-manager --add-repo https://example.com/updates/x86_64

dnf config-manager --dump   - show actual config