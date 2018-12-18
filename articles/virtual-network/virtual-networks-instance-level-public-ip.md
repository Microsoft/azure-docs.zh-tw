---
title: Azure 執行個體層級公用 IP (傳統) 位址 | Microsoft Docs
description: 了解執行個體層級公用 IP (ILPIP) 位址，以及使用 PowerShell 來管理它們的方式。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: tysonn
ms.assetid: 07eef6ec-7dfe-4c4d-a2c2-be0abfb48ec5
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/03/2018
ms.author: genli
ms.openlocfilehash: 7d8325ce04a9fa7853fb622062022a6938375f96
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47430976"
---
# <a name="instance-level-public-ip-classic-overview"></a>執行個體層級公用 IP (Classic) 概觀
執行個體層級公用 IP (ILPIP) 是您可以直接指派至 VM 或雲端服務角色執行個體的公用 IP 位址，而不是指派至 VM 或角色執行個體所在的雲端服務。 ILPIP 不會取代指派給雲端服務的虛擬 IP (VIP)。 應該說是您可以用來直接連接到 VM 或角色執行個體的其他 IP 位址。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議您透過 Resource Manager 建立 VM。 請確定您了解 [IP 位址](virtual-network-ip-addresses-overview-classic.md) 在 Azure 中的運作方式。

![ILPIP 和 VIP 之間的差異](./media/virtual-networks-instance-level-public-ip/Figure1.png)

如圖 1 所示，儘管通常是使用 VIP:&lt;連接埠號碼&gt; 來存取個別 VM，但還是會使用 VIP 來存取雲端服務。 將 ILPIP 指派給特定的 VM，就能直接使用該 IP 位址來存取 VM。

當您在 Azure 中建立雲端服務時，對應的 DNS A 記錄即會自動建立，以允許透過完整格式的網域名稱 (FQDN) 存取服務，而不需使用實際的 VIP。 相同程序也適用於 ILPIP，但可改為透過 FQDN 而不是 ILPIP 來允許存取 VM 或角色執行個體。 例如，若您建立名為 *contosoadservice* 的雲端服務，並設定名為 *contosoweb* 且具有兩個執行個體的 Web 角色，且 .cscfg 中的 `domainNameLabel` 設定為 *WebPublicIP*，Azure 將會為那些執行個體註冊下列 A 記錄：


* WebPublicIP.0.contosoadservice.cloudapp.net
* WebPublicIP.1.contosoadservice.cloudapp.net
* ...


> [!NOTE]
> 您只能針對每個 VM 或角色執行個體指派一個 ILPIP。 您可以針對每個訂用帳戶最多使用 5 個 ILPIP。 ILPIP 不支援多個 NIC VM。
> 
> 

## <a name="why-would-i-request-an-ilpip"></a>為什麼我要要求 ILPIP？
如果想要透過直接指派 IP 位址的方式連接到 VM 或角色執行個體，而不是使用雲端服務 VIP:&lt;連接埠號碼&gt;，請為 VM 或角色執行個體要求 ILPIP。

* **主動式 FTP**：透過將 ILPIP 指派給 VM，VM 就可以在所有的連接埠上接收流量。 VM 不需要端點就可以接收流量。  如需 FTP 通訊協定的詳細資訊，請參閱 [FTP 通訊協定概觀] (https://en.wikipedia.org/wiki/File_Transfer_Protocol#Protocol_overview)。
* **輸出 IP**：源自 VM 的輸出流量會對應至 ILPIP，因為來源與 ILPIP 可向外部實體唯一識別 VM。

> [!NOTE]
> ILPIP 過去被稱為公用 IP (PIP) 位址。
> 

## <a name="manage-an-ilpip-for-a-vm"></a>管理 VM 的 ILPIP
下列工作可讓您建立、指派和移除 VM 的 ILPIP：

### <a name="how-to-request-an-ilpip-during-vm-creation-using-powershell"></a>如何在建立 VM 期間使用 PowerShell 要求 ILPIP
下列 PowerShell 指令碼會建立名為 *FTPService* 的雲端服務、從 Azure 擷取映像、使用擷取的映像建立名為 *FTPInstance* 的 VM、設定要使用 ILPIP 的 VM，以及將 VM 加入新服務：

```powershell
New-AzureService -ServiceName FTPService -Location "Central US"

$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}

#Set "current" storage account for the subscription. It will be used as the location of new VM disk

Set-AzureSubscription -SubscriptionName <SubName> -CurrentStorageAccountName <StorageAccountName>

#Create a new VM configuration object

New-AzureVMConfig -Name FTPInstance -InstanceSize Small -ImageName $image.ImageName `
| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
| Set-AzurePublicIP -PublicIPName ftpip | New-AzureVM -ServiceName FTPService -Location "Central US"

```
如果您想要指定其他儲存體帳戶作為新的 VM 磁碟位置，您可以使用 **MediaLocation** 參數：

```powershell
    New-AzureVMConfig -Name FTPInstance -InstanceSize Small -ImageName $image.ImageName `
     -MediaLocation https://management.core.windows.net/<SubscriptionID>/services/storageservices/<StorageAccountName> `
    | Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
    | Set-AzurePublicIP -PublicIPName ftpip | New-AzureVM -ServiceName FTPService -Location "Central US"
```

### <a name="how-to-retrieve-ilpip-information-for-a-vm"></a>如何擷取 VM 的 ILPIP 資訊
若要檢視使用上述指令碼所建立 VM 的 ILPIP 資訊，請執行下列 PowerShell 命令，並觀察 *PublicIPAddress* 和 *PublicIPName* 的值：

```powershell
Get-AzureVM -Name FTPInstance -ServiceName FTPService
```

預期的輸出：
 
    DeploymentName              : FTPService
    Name                        : FTPInstance
    Label                       : 
    VM                          : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVM
    InstanceStatus              : ReadyRole
    IpAddress                   : 100.74.118.91
    InstanceStateDetails        : 
    PowerState                  : Started
    InstanceErrorCode           : 
    InstanceFaultDomain         : 0
    InstanceName                : FTPInstance
    InstanceUpgradeDomain       : 0
    InstanceSize                : Small
    HostName                    : FTPInstance
    AvailabilitySetName         : 
    DNSName                     : http://ftpservice888.cloudapp.net/
    Status                      : ReadyRole
    GuestAgentStatus            :   Microsoft.WindowsAzure.Commands.ServiceManagement.Model.GuestAgentStatus
    ResourceExtensionStatusList : {Microsoft.Compute.BGInfo}
    PublicIPAddress             : 104.43.142.188
    PublicIPName                : ftpip
    NetworkInterfaces           : {}
    ServiceName                 : FTPService
    OperationDescription        : Get-AzureVM
    OperationId                 : 568d88d2be7c98f4bbb875e4d823718e
    OperationStatus             : OK

### <a name="how-to-remove-an-ilpip-from-a-vm"></a>如何從 VM 移除 ILPIP
若要移除在上述指令碼中加入 VM 的 ILPIP，請執行下列 PowerShell 命令：

```powershell
Get-AzureVM -ServiceName FTPService -Name FTPInstance | Remove-AzurePublicIP | Update-AzureVM
```

### <a name="how-to-add-an-ilpip-to-an-existing-vm"></a>如何將 ILPIP 加入現有的 VM
若要將 ILPIP 加入至使用上述指令碼建立的 VM，請執行下列命令：

```powershell
Get-AzureVM -ServiceName FTPService -Name FTPInstance | Set-AzurePublicIP -PublicIPName ftpip2 | Update-AzureVM
```

## <a name="manage-an-ilpip-for-a-cloud-services-role-instance"></a>管理雲端服務角色執行個體的 ILPIP

若要將 ILPIP 加入至雲端服務角色執行個體，請完成下列步驟：

1. 完成[如何設定雲端服務](../cloud-services/cloud-services-how-to-configure-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#reconfigure-your-cscfg)文章中的步驟，以下載雲端服務的 .cscfg 檔案。
2. 加入 `InstanceAddress` 元素來更新 .cscfg 檔案。 下列範例會加入一個名稱為 *MyPublicIP* 的 ILPIP 到名稱為 *WebRole1* 的角色執行個體： 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ServiceConfiguration serviceName="ILPIPSample" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="4" osVersion="*" schemaVersion="2014-01.2.3">
      <Role name="WebRole1">
        <Instances count="1" />
          <ConfigurationSettings>
        <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
          </ConfigurationSettings>
      </Role>
      <NetworkConfiguration>
        <AddressAssignments>
          <InstanceAddress roleName="WebRole1">
        <PublicIPs>
          <PublicIP name="MyPublicIP" domainNameLabel="WebPublicIP" />
            </PublicIPs>
          </InstanceAddress>
        </AddressAssignments>
      </NetworkConfiguration>
    </ServiceConfiguration>
    ```
3. 完成[如何設定雲端服務](../cloud-services/cloud-services-how-to-configure-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#reconfigure-your-cscfg)文章中的步驟，以上傳雲端服務的 .cscfg 檔案。

### <a name="how-to-retrieve-ilpip-information-for-a-cloud-service"></a>如何擷取雲端服務的 ILPIP 資訊
若要檢視每個角色執行個體的 ILPIP 資訊，請執行下列 PowerShell 命令，並觀察 *PublicIPAddress*、*PublicIPName*、*PublicIPDomainNameLabel* 和 *PublicIPFqdns* 的值：

```powershell
Add-AzureAccount

$roles = Get-AzureRole -ServiceName <Cloud Service Name> -Slot Production -RoleName WebRole1 -InstanceDetails

$roles[0].PublicIPAddress
$roles[1].PublicIPAddress
```

您也可以使用 `nslookup` 來查詢子網域的 A 記錄：

```batch
nslookup WebPublicIP.0.<Cloud Service Name>.cloudapp.net
``` 

## <a name="next-steps"></a>後續步驟
* 了解 [IP 位址](virtual-network-ip-addresses-overview-classic.md) 在傳統部署模型中的運作方式。
* 深入了解 [保留的 IP](virtual-networks-reserved-public-ip.md)。
