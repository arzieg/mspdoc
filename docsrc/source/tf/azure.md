# Azure

## Default Subscription ändern


``az account list --output table``    // alle accounts auflisten

``az account show --output table``    // Default account

``az account set --subscription "<subscriptionID>"``   // default account setzen


## Anmelden an azure: 

`az login`

über URL Link 

`az login --use-device-code`


# AZcopy installieren

https://learn.microsoft.com/de-de/azure/storage/common/storage-use-azcopy-v10?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json&tabs=dnf

```
curl -sSL -O https://packages.microsoft.com/config/ubuntu/24.04/
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install azcopy
```

