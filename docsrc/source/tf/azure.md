# Azure

## Default Subscription ändern


``az account list --output table``    // alle accounts auflisten

``az account show --output table``    // Default account

``az account set --subscription "<subscriptionID>"``   // default account setzen


## Anmelden an azure: 

`az login`

über URL Link 

`az login --use-device-code`


## Rollen 

az role assignment list --assignee \<email\> --include-groups --output table

### Rolle auf eine spezielle Ressource

az role assignment list --scope /subscriptions/\<id\>/resourceGroups/.../providers/Microsoft.Storage/storageAccounts/\<storageaccount\> --assignee \<mail\> --include-groups --output table

# AZcopy installieren

https://learn.microsoft.com/de-de/azure/storage/common/storage-use-azcopy-v10?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json&tabs=dnf

```
curl -sSL -O https://packages.microsoft.com/config/ubuntu/24.04/
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install azcopy
```

# VM Typen

```
az vm list-sizes --location "Germany West Central" --query "[?supportsGen2==true]" -o table

Standard_B2s
```

Show #nic's, #cpu, #ram, #generation
```
az vm list-skus --location "germanywestcentral" --query "[].{Name:name, vCPUs:capabilities[?name=='vCPUs'].value | [0], MemoryGB:capabilities[?name=='MemoryGB'].value | [0], MaxNICs:capabilities[?name=='MaxNetworkInterfaces'].value | [0]}" --output table

```

Filter for 4 NIC's
```
az vm list-skus --location "germanywestcentral" --query "[?capabilities[?name=='MaxNetworkInterfaces'].value | [0] == '4'].{Name:name, vCPUs:capabilities[?name=='vCPUs'].value | [0], MemoryGB:capabilities[?name=='MemoryGB'].value | [0], MaxNICs:capabilities[?name=='MaxNetworkInterfaces'].value | [0], Generation:capabilities[?name=='Generation'].value | [0]}" --output table
```




# VM Namenskonventionen 

https://learn.microsoft.com/en-us/azure/virtual-machines/vm-naming-conventions

[Family] + [Sub-family]* + [# of vCPUs] + [Constrained vCPUs]* + [Additive Features] + [Accelerator Type]* + [Version]

|Value	| Explanation|
|-------|------------|
|Family	|Indicates the VM Family Series|
|*Subfamily|	Used for specialized VM differentiations only|
|# of vCPUs	|Denotes the number of vCPUs of the VM|
|*Constrained vCPUs|Used for certain VM sizes only. Denotes the number of vCPUs for the constrained vCPU capable size|
|Additive Features|	Lower case letters denote additive features, such as:|
||a = AMD-based processor|
||b = Block Storage performance|
||d = diskful (that is, a local temp disk is present); this feature is for newer Azure VMs, see Ddv4 and Ddsv4-series|
||i = isolated size|
||l = low memory; a lower amount of memory than the memory intensive size|
||m = memory intensive; the most amount of memory in a particular size|
||p = ARM Cpu|
||t = tiny memory; the smallest amount of memory in a particular size|
||s = Premium Storage capable, including possible use of Ultra SSD (Note: some newer sizes without the attribute of s can still support Premium Storage, such as M128, M64, etc.)|
||C = Confidential|
||NP = node packing|
|*Accelerator Type|	Denotes the type of hardware accelerator in the specialized/GPU SKUs. Only the new specialized/GPU SKUs launched from Q3 2020 have the hardware accelerator in the name.|
|Version|	Denotes the version of the VM Family Series|



# Serielle Konsole

1. Serielle Konsole öffen und neustarten

Darauf achten, dass man mit der Maus in der Console ist und diese aktiviert ist. 
Während Neustart 
* ESC drücken bei UEFI um in EFI Menü zu gelangen
* Shift bei BIOS 

