---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: msraiye
ms.service: virtual-machines
ms.topic: include
ms.date: 6/8/2018
ms.author: raiye
ms.custom: include file
ms.openlocfilehash: a7fe2cf151b79b02f4f8996ad938d8fc262a5f77
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49400175"
---
# <a name="enable-write-accelerator"></a>啟用寫入加速器

寫入加速器在具有 Azure 受控磁碟的進階儲存體上，是 M 系列虛擬機器 (VM) 專屬的磁碟功能。 如同名稱所示，這個功能的目的是針對 Azure 進階儲存體改善寫入的 I/O 延遲。 需要記錄檔更新，才能以高效能的方式保存到新式資料庫的磁碟時，寫入加速器是最理想的選擇。

寫入加速器以全面推出，供公用雲端中的 M 系列 VM 使用。

## <a name="planning-for-using-write-accelerator"></a>規劃使用寫入加速器

寫入加速器應該用於包含 DBMS 之交易記錄或重做記錄的磁碟區。 不建議將寫入加速器用於 DBMS 的資料磁碟區，因為此功能是針對在記錄磁碟使用而最佳化。

寫入加速器只能與 [Azure 受控磁碟](https://azure.microsoft.com/services/managed-disks/)搭配運作。

> [!IMPORTANT]
> 為 VM 的作業系統磁碟啟用寫入加速器時，會重新啟動此 VM。
>
> 若要啟用現有 Azure 磁碟的寫入加速器，且此磁碟「不」屬於使用 Windows 磁碟或磁碟區管理員、Windows 儲存空間、Windows 相應放大檔案伺服器 (SOFS)、Linux LVM 或 MDADM，從多個磁碟建置的磁碟區，則必須關閉存取此 Azure 磁碟的工作負載。 「必須」關閉使用 Azure 磁碟的資料庫應用程式。
>
> 如果您想要為現有磁碟區啟用或停用寫入加速器，而此磁碟區是使用 Windows 磁碟或磁碟區管理員、Windows 儲存空間、Windows 相應放大檔案伺服器 (SOFS)、Linux LVM 或 MDADM，從多個 Azure 進階儲存體磁碟建置並設定為等量磁碟區，則建置磁碟區的所有磁碟都必須個別使用不同的步驟啟用或停用寫入加速器。 **啟用或停用此種設定的寫入加速器之前，請先關閉 Azure VM**。

針對 SAP 相關虛擬機器組態，應該沒有必要為 OS 磁碟啟用寫入加速器。

### <a name="restrictions-when-using-write-accelerator"></a>使用寫入加速器時的限制

針對 Azure 磁碟/VHD 使用寫入加速器時，適用以下限制：

- 必須將進階磁碟快取設定為「無」或「唯讀」。 不支援所有其他的快取模式。
- 尚不支援對已啟用寫入加速器的磁碟使用快照集。 這項限制會封鎖「Azure 備份服務」對虛擬機器的所有磁碟執行應用程式一致快照的能力。
- 只有較小的 I/O 大小 (<=32 KiB) 會採用加速路徑。 在大量載入資料或在不同 DBMS 的交易記錄緩衝區保存到儲存體之前就達到較大填滿程度的工作負載情況下，寫入磁碟的 I/O 可能不會採用加速路徑。

寫入加速器可以支援之每個 VM 的 Azure 進階儲存體 VHD 有其上限。 目前的上限是：

| VM SKU | 寫入加速器磁碟的數目 | 每個 VM 的寫入加速器磁碟 IOPS |
| --- | --- | --- |
| M128ms，128s | 16 | 8000 |
| M64ms，M64ls，M64s | 8 | 4000 |
| M32ms，M32ls，M32ts，M32s | 4 | 2000 |
| M16ms，M16s | 2 | 1000 |
| M8ms，M8s | 1 | 500 |

IOPS 限制是針對每部虛擬機器而「不是」每個磁碟。 所有寫入加速器磁碟都會共用相同的每部虛擬機器 IOPS 限制。

## <a name="enabling-write-accelerator-on-a-specific-disk"></a>在特定磁碟上啟用寫入加速器

接下來的幾節將說明如何在 Azure 進階儲存體 VHD 上啟用寫入加速器。

### <a name="prerequisites"></a>必要條件

以下先決條件適用於本文撰寫時的寫入加速器使用：

- 您想要套用 Azure 寫入加速器的磁碟必須為進階儲存體上的 [Azure 受控磁碟](https://azure.microsoft.com/services/managed-disks/)。
- 您必須使用 M 系列 VM

## <a name="enabling-azure-write-accelerator-using-azure-powershell"></a>使用 Azure PowerShell 啟用 Azure 寫入加速器

5.5.0 版之後的 Azure PowerShell 模組皆已引進相關 Cmdlet 的變更，可以為特定 Azure 進階儲存體磁碟啟用或停用寫入加速器。
為了要啟用或部署寫入加速器所支援的磁碟，下列 PowerShell 命令已經過變更，並擴大為接受寫入加速器的參數。

下列 Cmdlet 中已加入了新的開關參數 **-WriteAccelerator**：

- [Set-AzureRmVMOsDisk](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmosdisk?view=azurermps-6.0.0)
- [Add-AzureRmVMDataDisk](https://docs.microsoft.com/powershell/module/AzureRM.Compute/Add-AzureRmVMDataDisk?view=azurermps-6.0.0)
- [Set-AzureRmVMDataDisk](https://docs.microsoft.com/powershell/module/AzureRM.Compute/Set-AzureRmVMDataDisk?view=azurermps-6.0.0)
- [Add-AzureRmVmssDataDisk](https://docs.microsoft.com/powershell/module/AzureRM.Compute/Add-AzureRmVmssDataDisk?view=azurermps-6.0.0)

未提供參數等於將屬性設為 false，而且會部署沒有寫入加速器支援的磁碟。

下列 Cmdlet 已加入了新的開關參數 **-OsDiskWriteAccelerator**：

- [Set-AzureRmVmssStorageProfile](https://docs.microsoft.com/powershell/module/AzureRM.Compute/Set-AzureRmVmssStorageProfile?view=azurermps-6.0.0)

未指定參數等於將屬性設為 false (預設)，而且會傳回不利用寫入加速器的磁碟。

下列 Cmdlet 已加入了新的選擇性布林值 (不可使用 null 值) 參數 **-OsDiskWriteAccelerator**：

- [Update-AzureRmVM](https://docs.microsoft.com/powershell/module/AzureRM.Compute/Update-AzureRmVM?view=azurermps-6.0.0)
- [Update-AzureRmVmss](https://docs.microsoft.com/powershell/module/AzureRM.Compute/Update-AzureRmVmss?view=azurermps-6.0.0)

指定 $true 或 $false 來控制磁碟的 Azure 寫入加速器支援。

命令範例如下︰

```PowerShell
New-AzureRmVMConfig | Set-AzureRmVMOsDisk | Add-AzureRmVMDataDisk -Name "datadisk1" | Add-AzureRmVMDataDisk -Name "logdisk1" -WriteAccelerator | New-AzureRmVM

Get-AzureRmVM | Update-AzureRmVM -OsDiskWriteAccelerator $true

New-AzureRmVmssConfig | Set-AzureRmVmssStorageProfile -OsDiskWriteAccelerator | Add-AzureRmVmssDataDisk -Name "datadisk1" -WriteAccelerator:$false | Add-AzureRmVmssDataDisk -Name "logdisk1" -WriteAccelerator | New-AzureRmVmss

Get-AzureRmVmss | Update-AzureRmVmss -OsDiskWriteAccelerator:$false
```

後面幾節有兩個主要案例的指令碼。

### <a name="adding-a-new-disk-supported-by-write-accelerator-using-powershell"></a>使用 PowerShell 新增寫入加速器所支援的新磁碟

您可以使用這個指令碼將新的磁碟新增至您的 VM。 使用此指令碼所建立的磁碟將會使用寫入加速器。

以適用您特定部署的值取代 `myVM`、`myWAVMs`、`log001`、磁碟大小和磁碟 LunID。

```PowerShell
# Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "log001"
#LUN Id
$lunid=8
#size
$size=1023
#Pulls the VM info for later
$vm=Get-AzurermVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Add-AzureRmVMDataDisk -CreateOption empty -DiskSizeInGB $size -Name $vmname-$datadiskname -VM $vm -Caching None -WriteAccelerator:$true -lun $lunid
#Updates the VM with the disk config - does not require a reboot
Update-AzureRmVM -ResourceGroupName $rgname -VM $vm
```

### <a name="enabling-write-accelerator-on-an-existing-azure-disk-using-powershell"></a>使用 PowerShell 在現有 Azure 磁碟上啟用寫入加速器

您可以使用以下指令在現有磁碟上啟用寫入加速器。 以適用於您特定部署的值取代 `myVM`、`myWAVMs` 和 `test-log001`。 指令碼會在 **$newstatus** 的值設定為 '$true' 時，將寫入加速器新增至現有磁碟。 若使用 '$false' 值，則會停用指定磁碟的寫入加速器。

```PowerShell
#Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "test-log001" 
#new Write Accelerator status ($true for enabled, $false for disabled) 
$newstatus = $true
#Pulls the VM info for later
$vm=Get-AzurermVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Set-AzureRmVMDataDisk -VM $vm -Name $datadiskname -Caching None -WriteAccelerator:$newstatus
#Updates the VM with the disk config - does not require a reboot
Update-AzureRmVM -ResourceGroupName $rgname -VM $vm
```

> [!Note]
> 執行上述指令碼會將指定的磁碟中斷連結，對磁碟啟用寫入加速器，然後再重新連結磁碟

## <a name="enabling-write-accelerator-using-the-azure-portal"></a>使用 Azure 入口網站啟用寫入加速器

您可以透過讓您指定磁碟快取設定所在的入口網站，啟用寫入加速器：

![Azure 入口網站上的寫入加速器](./media/virtual-machines-common-how-to-enable-write-accelerator/wa_scrnsht.png)

## <a name="enabling-write-accelerator-using-the-azure-cli"></a>使用 Azure CLI 啟用寫入加速器

您可以使用 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) 啟用寫入加速器。

若要在現有磁碟上啟用寫入加速器，請使用 [az vm update](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-update)，如果您將 diskName、VMName 和 ResourceGroup 取代為您自己的值，可以使用下列範例：`az vm update -g group1 -n vm1 -write-accelerator 1=true`

若要將磁碟與已啟用寫入加速器連結，請使用 [az vm disk attach](https://docs.microsoft.com/cli/azure/vm/disk?view=azure-cli-latest#az-vm-disk-attach)，如果您要取代為您自己的值，可以使用下列範例：`az vm disk attach -g group1 -vm-name vm1 -disk d1 --enable-write-accelerator`

若要停用寫入加速器，請使用 [az vm update](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-update)，將屬性設定為 false：`az vm update -g group1 -n vm1 -write-accelerator 0=false 1=false`

## <a name="enabling-write-accelerator-using-rest-apis"></a>使用 Rest API 啟用寫入加速器

若要透過 Azure Rest API 部署，您需要安裝 Azure ARMClient。

### <a name="install-armclient"></a>安裝 ARMClient

若要執行 ARMClient，您需要透過 Chocolatey 安裝它。 您可以使用 cmd.exe 或 PowerShell 來安裝它。 以提高的權限使用這些命令 (「以系統管理員身分執行」)。

使用 cmd.exe 執行下列命令：`@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`

使用 Power Shell 執行下列命令：`Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`

現在您可以使用下列命令在 cmd.exe 或 PowerShell 安裝 ARMClient：`choco install armclient`

### <a name="getting-your-current-vm-configuration"></a>取得目前的 VM 設定

若要變更磁碟設定的屬性，首先要取得 JSON 檔案中的目前設定。 執行下列命令可以取得目前的設定：`armclient GET /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 > <<filename.json>>`

將 '<<   >>' 括住的字詞取代為您的資料，包括 JSON 檔案應有的檔案名稱。

輸出看起來像這樣：

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"

```

下一步，更新 JSON 檔案，並在名為 'log1' 的磁碟上啟用寫入加速器。 將這個屬性新增至 JSON 檔案中磁碟的快取項目之後，即可完成此步驟。

```JSON
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
```

然後使用此命令更新現有的部署：`armclient PUT /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 @<<filename.json>>`

輸出看起來應該會像下面這樣。 您可以看到有一個磁碟已啟用寫入加速器。

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"
```

執行此變更之後，寫入加速器應該要支援磁碟機。
