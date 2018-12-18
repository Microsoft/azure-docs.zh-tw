---
title: 靜態內部私人 IP - Azure VM - 傳統
description: 了解靜態內部 IP (DIP) 以及如何管理
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: tysonn
ms.assetid: 93444c6f-af1b-41f8-a035-77f5c0302bf0
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/18/2018
ms.author: genli
ms.openlocfilehash: 661d2f789ace8da68b6d65609d4584a11967a01f
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34366608"
---
# <a name="how-to-set-a-static-internal-private-ip-address-using-powershell-classic"></a>如何使用 PowerShell 設定靜態內部私人 IP 位址 (傳統)
在大部分情況下，您不需要針對虛擬機器指定靜態內部 IP 位址。 虛擬網路中的 VM 會從您指定的範圍自動接收內部 IP 位址。 但在某些情況下，針對特定 VM 指定靜態 IP 位址是合理的。 例如，如果您的 VM 即將執行 DNS 或將成為網域控制站。 靜態內部 IP 位址會伴隨 VM 而存在，甚至是透過停止/取消佈建狀態。 

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 [Resource Manager 部署模型](virtual-networks-static-private-ip-arm-ps.md)。
> 
> 

## <a name="how-to-verify-if-a-specific-ip-address-is-available"></a>如何驗證特定 IP 位址是否可用
若要驗證 IP 位址 10.0.0.7 在名為 TestVnet 的 VNet 中是否可用，請執行下列 PowerShell 命令，並驗證 IsAvailable 的值：

    Test-AzureStaticVNetIP –VNetName TestVNet –IPAddress 10.0.0.7 

    IsAvailable          : True
    AvailableAddresses   : {}
    OperationDescription : Test-AzureStaticVNetIP
    OperationId          : fd3097e1-5f4b-9cac-8afa-bba1e3492609
    OperationStatus      : Succeeded

> [!NOTE]
> 如果您想要在安全的環境中測試上述命令，請依照[建立虛擬網路 (傳統)](virtual-networks-create-vnet-classic-pportal.md) 中的指導方針，建立名為 *TestVnet* 的 VNet，並確保它使用 *10.0.0.0/8* 位址空間。
> 
> 

## <a name="how-to-specify-a-static-internal-ip-when-creating-a-vm"></a>如何在建立 VM 時指定靜態內部 IP
下方 PowerShell 指令碼會建立名為 TestService 的新雲端服務，接著從 Azure 中擷取映像，然後在新的雲端服務中使用擷取的映像建立名為 TestVM 的 VM，接下來設定 VM 位於稱為 Subnet-1 子網路中，並設定 10.0.0.7 作為 VM 的靜態內部 IP：

    New-AzureService -ServiceName TestService -Location "Central US"
    $image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
    New-AzureVMConfig -Name TestVM -InstanceSize Small -ImageName $image.ImageName `
    | Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
    | Set-AzureSubnet –SubnetNames Subnet-1 `
    | Set-AzureStaticVNetIP -IPAddress 10.0.0.7 `
    | New-AzureVM -ServiceName "TestService" –VNetName TestVnet

## <a name="how-to-retrieve-static-internal-ip-information-for-a-vm"></a>如何擷取 VM 的靜態內部 IP 資訊
若要檢視使用上述指令碼建立之 VM 的 IP 資訊，請執行下列 PowerShell 命令，並觀察 *IpAddress*的值：

    Get-AzureVM -Name TestVM -ServiceName TestService

    DeploymentName              : TestService
    Name                        : TestVM
    Label                       : 
    VM                          : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVM
    InstanceStatus              : Provisioning
    IpAddress                   : 10.0.0.7
    InstanceStateDetails        : Windows is preparing your computer for first use...
    PowerState                  : Started
    InstanceErrorCode           : 
    InstanceFaultDomain         : 0
    InstanceName                : TestVM
    InstanceUpgradeDomain       : 0
    InstanceSize                : Small
    HostName                    : rsR2-797
    AvailabilitySetName         : 
    DNSName                     : http://testservice000.cloudapp.net/
    Status                      : Provisioning
    GuestAgentStatus            : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.GuestAgentStatus
    ResourceExtensionStatusList : {Microsoft.Compute.BGInfo}
    PublicIPAddress             : 
    PublicIPName                : 
    NetworkInterfaces           : {}
    ServiceName                 : TestService
    OperationDescription        : Get-AzureVM
    OperationId                 : 34c1560a62f0901ab75cde4fed8e8bd1
    OperationStatus             : OK

## <a name="how-to-remove-a-static-internal-ip-from-a-vm"></a>如何從 VM 移除靜態內部 IP
若要移除在上述指令碼中新增至 VM 的靜態內部 IP，請執行下列 PowerShell 命令：

    Get-AzureVM -ServiceName TestService -Name TestVM `
    | Remove-AzureStaticVNetIP `
    | Update-AzureVM

## <a name="how-to-add-a-static-internal-ip-to-an-existing-vm"></a>如何將靜態內部 IP 位址新增至現有的 VM
若要將靜態內部 IP 新增至使用上述指令碼建立的 VM，請執行下列命令：

    Get-AzureVM -ServiceName TestService000 -Name TestVM `
    | Set-AzureStaticVNetIP -IPAddress 10.10.0.7 `
    | Update-AzureVM

## <a name="next-steps"></a>後續步驟
[保留的 IP](virtual-networks-reserved-public-ip.md)

[執行個體層級公用 IP (ILPIP)](virtual-networks-instance-level-public-ip.md)

[保留的 IP REST API](https://msdn.microsoft.com/library/azure/dn722420.aspx)

