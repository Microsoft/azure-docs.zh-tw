---
title: 設定 VM (傳統) 的私人 IP 位址 - Azure PowerShell | Microsoft Docs
description: 了解如何使用 PowerShell 設定虛擬機器 (傳統) 的私人 IP 位址。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: tysonn
tags: azure-service-management
ms.assetid: 60c7b489-46ae-48af-a453-2b429a474afd
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/02/2016
ms.author: genli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: f99e67341d46e858cee7dd6a22f16fe06ad5b88a
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38678590"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-classic-using-powershell"></a>使用 PowerShell 設定虛擬機器 (傳統) 的私人 IP 位址

[!INCLUDE [virtual-networks-static-private-ip-selectors-classic-include](../../includes/virtual-networks-static-private-ip-selectors-classic-include.md)]

[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文涵蓋之內容包括傳統部署模型。 您也可以 [管理資源管理員部署模型中的靜態私人 IP 位址](virtual-networks-static-private-ip-arm-ps.md)。

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

以下的範例 PowerShell 命令會預期已經建立簡單的環境。 如果您想要執行如本文件中所顯示的命令，請先建置 [建立 VNet](virtual-networks-create-vnet-classic-netcfg-ps.md)中所說明的測試環境。

## <a name="how-to-verify-if-a-specific-ip-address-is-available"></a>如何驗證特定 IP 位址是否可用
若要驗證 IP 位址 192.168.1.101 在名為 TestVnet 的 VNet 中是否可用，請執行下列 PowerShell 命令，並驗證 IsAvailable 的值：

    Test-AzureStaticVNetIP –VNetName TestVNet –IPAddress 192.168.1.101 

預期的輸出：

    IsAvailable          : True
    AvailableAddresses   : {}
    OperationDescription : Test-AzureStaticVNetIP
    OperationId          : fd3097e1-5f4b-9cac-8afa-bba1e3492609
    OperationStatus      : Succeeded

## <a name="how-to-specify-a-static-private-ip-address-when-creating-a-vm"></a>建立 VM 時如何指定靜態私人 IP 位址
下方 PowerShell 指令碼會建立名為 TestService 的新雲端服務，接著從 Azure 中擷取映像，然後在新的雲端服務中使用擷取的映像建立名為 DNS01 的 VM，接下來設定 VM 位於 FrontEnd 子網路中，並設定 192.168.1.7 作為 VM 的靜態私人 IP：

    New-AzureService -ServiceName TestService -Location "Central US"
    $image = Get-AzureVMImage | where {$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
    New-AzureVMConfig -Name DNS01 -InstanceSize Small -ImageName $image.ImageName |
      Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! |
      Set-AzureSubnet –SubnetNames FrontEnd |
      Set-AzureStaticVNetIP -IPAddress 192.168.1.7 |
      New-AzureVM -ServiceName TestService –VNetName TestVNet

預期的輸出：

    WARNING: No deployment found in service: 'TestService'.
    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    New-AzureService     fcf705f1-d902-011c-95c7-b690735e7412 Succeeded      
    New-AzureVM          3b99a86d-84f8-04e5-888e-b6fc3c73c4b9 Succeeded  

## <a name="how-to-retrieve-static-private-ip-address-information-for-a-vm"></a>如何擷取 VM 的靜態私人 IP 位址資訊
若要檢視使用上述指令碼建立之 VM 的私人 IP 位址資訊，請執行下列 PowerShell 命令，並觀察 *IpAddress*的值：

    Get-AzureVM -Name DNS01 -ServiceName TestService

預期的輸出：

    DeploymentName              : TestService
    Name                        : DNS01
    Label                       : 
    VM                          : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVM
    InstanceStatus              : Provisioning
    IpAddress                   : 192.168.1.7
    InstanceStateDetails        : Windows is preparing your computer for first use...
    PowerState                  : Started
    InstanceErrorCode           : 
    InstanceFaultDomain         : 0
    InstanceName                : DNS01
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

## <a name="how-to-remove-a-static-private-ip-address-from-a-vm"></a>如何移除 VM 的靜態私人 IP 位址
若要移除上述指令碼中新增至 VM 的靜態私人 IP 位址，請執行下列 PowerShell 命令：

    Get-AzureVM -ServiceName TestService -Name DNS01 |
      Remove-AzureStaticVNetIP |
      Update-AzureVM

預期的輸出：

    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    Update-AzureVM       052fa6f6-1483-0ede-a7bf-14f91f805483 Succeeded

## <a name="how-to-add-a-static-private-ip-address-to-an-existing-vm"></a>如何將靜態私人 IP 位址新增至現有的 VM
若要將靜態私人 IP 位址新增至使用上述指令碼建立之 VM，請執行下列命令：

    Get-AzureVM -ServiceName TestService -Name DNS01 |
      Set-AzureStaticVNetIP -IPAddress 192.168.1.7 |
      Update-AzureVM

預期的輸出：

    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    Update-AzureVM       77d8cae2-87e6-0ead-9738-7c7dae9810cb Succeeded 

## <a name="set-ip-addresses-within-the-operating-system"></a>設定作業系統內的 IP 位址

除非必要，建議您不要靜態指派虛擬機器作業系統內已指派給 Azure 虛擬機器的私人 IP。 如果您確實手動設定作業系統內的私人 IP 位址，請確保該位址與指派給 Azure VM 的私人 IP 位址相同，否則可能會失去與虛擬機器的連線。 請勿手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。

## <a name="next-steps"></a>後續步驟
* 深入了解 [保留的公用 IP](virtual-networks-reserved-public-ip.md) 位址。
* 深入了解 [執行個體層級公用 IP (ILPIP)](virtual-networks-instance-level-public-ip.md) 位址。
* 請參閱 [保留 IP REST API](https://msdn.microsoft.com/library/azure/dn722420.aspx)。

