---
title: 將雲端服務連接到自訂網域控制站 | Microsoft Docs
description: 了解如何使用 PowerShell 和 AD 網域延伸將 Web/背景工作角色連接到自訂 AD 網域
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 1e2d7c87-d254-4e7a-a832-67f84411ec95
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.author: jeconnoc
ms.openlocfilehash: b05e20b5c99c6f1b5b1bf93ca781ec97284fba79
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39004910"
---
# <a name="connecting-azure-cloud-services-roles-to-a-custom-ad-domain-controller-hosted-in-azure"></a>將 Azure 雲端服務角色連接到裝載於 Azure 中的自訂 AD 網域控制站
我們會先在 Azure 中設定虛擬網路 (VNet)。 接著再將 Active Directory 網域控制站 (裝載於 Azure 虛擬機器上) 加入 VNet。 下一步是將現有雲端服務角色加入預先建立的 VNet，然後將它們連接到網域控制站。

在開始之前，請將以下幾件事牢記在心：

1. 本教學課程使用 PowerShell，因此請確認您已安裝 Azure PowerShell 且已準備就緒。 如需設定 Azure PowerShell 的說明，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。
2. AD 網域控制站和 Web/背景工作角色執行個體必須位在 VNet 中。

請依本逐步指南作業，如果遇到任何問題，請在本文結尾處留言。 我們將會回覆您 (沒錯，我們真的會閱讀留言)。

雲端服務所參考的網路必須是**傳統虛擬網路**。

## <a name="create-a-virtual-network"></a>建立虛擬網路
您可以使用 Azure 入口網站或 PowerShell 在 Azure 中建立虛擬網路。 本教學課程會使用 PowerShell。 若要使用 Azure 入口網站建立虛擬網路，請參閱[建立虛擬網路](../virtual-network/quick-create-portal.md)。 本文涵蓋建立虛擬網路 (Resource Manager)，但是您必須建立適用於雲端服務的虛擬網路 (傳統)。 若要這樣做，在入口網站中，選取 [建立資源]、在 [搜尋] 方塊中輸入「虛擬網路」，然後按 **Enter**。 在搜尋結果的 [所有項目] 下方，選取 [虛擬網路]。 在 [選取部署模型] 下方，選取 [傳統]，然後選取 [建立]。 您接著可以依照文中的步驟進行。

```powershell
#Create Virtual Network

$vnetStr =
@"<?xml version="1.0" encoding="utf-8"?>
<NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
    <VirtualNetworkConfiguration>
    <VirtualNetworkSites>
        <VirtualNetworkSite name="[your-vnet-name]" Location="West US">
        <AddressSpace>
            <AddressPrefix>[your-address-prefix]</AddressPrefix>
        </AddressSpace>
        <Subnets>
            <Subnet name="[your-subnet-name]">
            <AddressPrefix>[your-subnet-range]</AddressPrefix>
            </Subnet>
        </Subnets>
        </VirtualNetworkSite>
    </VirtualNetworkSites>
    </VirtualNetworkConfiguration>
</NetworkConfiguration>
"@;

$vnetConfigPath = "<path-to-vnet-config>"
Set-AzureVNetConfig -ConfigurationPath $vnetConfigPath
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器
完成虛擬網路的設定後，您需要建立 AD 網域控制站。 在本教學課程中，我們會在 Azure 虛擬機器上設定 AD 網域控制站。

若要這樣做，請使用下列命令透過 PowerShell 建立虛擬機器：

```powershell
# Initialize variables
# VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.

$vnetname = '<your-vnet-name>'
$subnetname = '<your-subnet-name>'
$vmsvc1 = '<your-hosted-service>'
$vm1 = '<your-vm-name>'
$username = '<your-username>'
$password = '<your-password>'
$affgrp = '<your- affgrp>'

# Create a VM and add it to the Virtual Network

New-AzureQuickVM -Windows -ServiceName $vmsvc1 -Name $vm1 -ImageName $imgname -AdminUsername $username -Password $password -AffinityGroup $affgrp -SubnetNames $subnetname -VNetName $vnetname
```

## <a name="promote-your-virtual-machine-to-a-domain-controller"></a>將虛擬機器提升為網域控制站
若要將虛擬機器設定為 AD 網域控制站，您需要登入 VM 並進行設定。

若要登入 VM，您可以透過 PowerShell 取得 RDP 檔案，請使用下列命令：

```powershell
# Get RDP file
Get-AzureRemoteDesktopFile -ServiceName $vmsvc1 -Name $vm1 -LocalPath <rdp-file-path>
```

登入 VM 後，請遵循[如何設定客戶的 AD 網域控制站](http://social.technet.microsoft.com/wiki/contents/articles/12370.windows-server-2012-set-up-your-first-domain-controller-step-by-step.aspx)中的逐步指南，將虛擬機器設為 AD 網域控制站。

## <a name="add-your-cloud-service-to-the-virtual-network"></a>將雲端服務加入虛擬網路
接下來，您需要將雲端服務部署新增至新的 VNet。 若要這樣做，請使用 Visual Studio 或選擇的編輯器將相關區段加入 cscfg，藉此修改雲端服務 cscfg。

```xml
<ServiceConfiguration serviceName="[hosted-service-name]" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="[os-family]" osVersion="*">
    <Role name="[role-name]">
    <Instances count="[number-of-instances]" />
    </Role>
    <NetworkConfiguration>

    <!--optional-->
    <Dns>
        <DnsServers><DnsServer name="[dns-server-name]" IPAddress="[ip-address]" /></DnsServers>
    </Dns>
    <!--optional-->

    <!--VNet settings
        VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.-->
    <VirtualNetworkSite name="[virtual-network-name]" />
    <AddressAssignments>
        <InstanceAddress roleName="[role-name]">
        <Subnets>
            <Subnet name="[subnet-name]" />
        </Subnets>
        </InstanceAddress>
    </AddressAssignments>
    <!--VNet settings-->

    </NetworkConfiguration>
</ServiceConfiguration>
```

接下來，請建置雲端服務專案並將它部署到 Azure。 如需將雲端服務封裝部署到 Azure 的說明，請參閱「 [如何建立和部署雲端服務](cloud-services-how-to-create-deploy-portal.md)

## <a name="connect-your-webworker-roles-to-the-domain"></a>將 Web/背景工作角色連接到網域
在 Azure 上部署雲端服務專案後，請使用 AD 網域延伸將角色執行個體連接到自訂 AD　網域。 若要將 AD 網域延伸加入現有雲端服務部署及加入自訂網域，請在 PowerShell 中執行下列命令：

```powershell
# Initialize domain variables

$domain = '<your-domain-name>'
$dmuser = '$domain\<your-username>'
$dmpswd = '<your-domain-password>'
$dmspwd = ConvertTo-SecureString $dmpswd -AsPlainText -Force
$dmcred = New-Object System.Management.Automation.PSCredential ($dmuser, $dmspwd)

# Add AD Domain Extension to the cloud service roles

Set-AzureServiceADDomainExtension -Service <your-cloud-service-hosted-service-name> -Role <your-role-name> -Slot <staging-or-production> -DomainName $domain -Credential $dmcred -JoinOption 35
```

就這麼簡單。

您的雲端服務應該已加入自訂網域控制站。 如果您想要深入了解設定 AD 網域延伸時可用的其他選項，請使用 PowerShell 說明。 以下是一些範例：

```powershell
help Set-AzureServiceADDomainExtension
help New-AzureServiceADDomainExtensionConfig
```
