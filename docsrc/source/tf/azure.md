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