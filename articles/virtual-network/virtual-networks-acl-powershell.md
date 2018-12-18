---
title: 管理 Azure 端點存取控制清單 | PowerShell | 傳統 | Microsoft Docs
description: 了解如何使用 PowerShell 管理 ACL
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: tysonn
ms.assetid: c84e40af-f351-4572-b3f0-d572d46bafe7
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
ms.author: jdial
ms.openlocfilehash: c3476908447380ccd7e8b9c0f1c2a55ae763cc1e
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23124886"
---
# <a name="manage-endpoint-access-control-lists-using-powershell-in-the-classic-deployment-model"></a>在傳統部署模型中使用 PowerShell 來管理端點存取控制清單
您可以使用 Azure PowerShell 或在管理入口網站中建立和管理端點的網路存取控制清單 (ACL)。 在本主題中，您會了解一些可使用 PowerShell 完成 ACL 一般工作的程序。 如需 Azure PowerShell Cmdlet 的清單，請參閱＜ [Azure 管理 Cmdlet](http://go.microsoft.com/fwlink/?LinkId=317721)＞。 如需有關 ACL 的詳細資訊，請參閱＜ [什麼是網路存取控制清單 (ACL)？](virtual-networks-acl.md)＞。 若您要使用管理入口網站來管理 ACL，請參閱[如何設定虛擬機器的端點](../virtual-machines/windows/classic/setup-endpoints.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

## <a name="manage-network-acls-by-using-azure-powershell"></a>使用 Azure PowerShell 來管理網路 ACL
您可以使用 Azure PowerShell Cmdlet 來建立、移除和設定 (Set) 網路存取控制清單 (ACL)。 我們已加入一些您可以使用 PowerShell 設定 ACL 方式的幾個範例。

若要擷取 ACL PowerShell Cmdlet 的完整清單，您可以使用下列其中一項：

    Get-Help *AzureACL*
    Get-Command -Noun AzureACLConfig

### <a name="create-a-network-acl-with-rules-that-permit-access-from-a-remote-subnet"></a>建立網路 ACL 搭配規則以允許從遠端子網路進行存取
下方範例示範如何建立包含規則的新 ACL。 此 ACL 接著會套用至虛擬機器端點。 下方範例中的 ACL 規則將允許從遠端子網路進行存取。 若要建立新的網路 ACL，並包含遠端子網路的允許規則，請開啟 Azure PowerShell ISE。 複製並貼上下方的指令碼，接著使用您自己的值設定指令碼後執行。

1. 建立新的網路 ACL 物件。
   
        $acl1 = New-AzureAclConfig
2. 設定規則以允許從遠端子網路進行存取。 在下方範例中，您可以將規則設定為 100 (其中的優先順序高於 200 及以上) 以允許遠端子網路 10.0.0.0/8 存取虛擬機器端點。 根據您自己的組態需求來取代值。 「SharePoint ACL config」的名稱應該取代為您命名此規則的易記名稱。
   
        Set-AzureAclConfig –AddRule –ACL $acl1 –Order 100 `
            –Action permit –RemoteSubnet "10.0.0.0/8" `
            –Description "SharePoint ACL config"
3. 如需其他規則，請重複執行 Cmdlet，並根據您自己的組態需求來取代值。 請務必變更規則編號「Order」以反映您想要套用規則的順序。 規則編號較低的優先順序高於較高的編號。
   
        Set-AzureAclConfig –AddRule –ACL $acl1 –Order 200 `
            –Action permit –RemoteSubnet "157.0.0.0/8" `
            –Description "web frontend ACL config"
4. 接下來，您可以建立新的端點 (Add)，或設定現有端點 (Set) 的 ACL。 在此範例中，我們將會新增稱為「web」的新虛擬機器端點，並使用 ACL 設定更新虛擬機器端點。
   
        Get-AzureVM –ServiceName $serviceName –Name $vmName `
        | Add-AzureEndpoint –Name "web" –Protocol tcp –Localport 80 - PublicPort 80 –ACL $acl1 `
        | Update-AzureVM
5. 接下來，結合 Cmdlet 並執行指令碼。 在此範例中，結合的 Cmdlet 如下所示：
   
        $acl1 = New-AzureAclConfig
        Set-AzureAclConfig –AddRule –ACL $acl1 –Order 100 `
            –Action permit –RemoteSubnet "10.0.0.0/8" `
            –Description "Sharepoint ACL config"
        Set-AzureAclConfig –AddRule –ACL $acl1 –Order 200 `
            –Action permit –RemoteSubnet "157.0.0.0/8" `
            –Description "web frontend ACL config"
        Get-AzureVM –ServiceName $serviceName –Name $vmName `
        |Add-AzureEndpoint –Name "web" –Protocol tcp –Localport 80 - PublicPort 80 –ACL $acl1 `
        |Update-AzureVM

### <a name="remove-a-network-acl-rule-that-permits-access-from-a-remote-subnet"></a>設定網路 ACL 規則以允許從遠端子網路進行存取
下方範例示範移除網路 ACL 規則的方式。  若要移除包含遠端子網路允許規則的網路 ACL 規則，請開啟 Azure PowerShell ISE。 複製並貼上下方的指令碼，接著使用您自己的值設定指令碼後執行。

1. 第一個步驟是取得虛擬機器端點的網路 ACL 物件， 然後移除 ACL 規則。 在此案例中，我們依據規則 ID 進行移除。 這只會從 ACL 移除規則 ID 0， 並不會從虛擬機器端點移除 ACL 物件。
   
        Get-AzureVM –ServiceName $serviceName –Name $vmName `
        | Get-AzureAclConfig –EndpointName "web" `
        | Set-AzureAclConfig –RemoveRule –ID 0 –ACL $acl1
2. 接下來，您必須將網路 ACL 物件套用至虛擬機器端點，並更新虛擬機器。
   
        Get-AzureVM –ServiceName $serviceName –Name $vmName `
        | Set-AzureEndpoint –ACL $acl1 –Name "web" `
        | Update-AzureVM

### <a name="remove-a-network-acl-from-a-virtual-machine-endpoint"></a>從虛擬機器端點移除網路 ACL
在某些情況下，您可能會想要從虛擬機器端點移除網路 ACL 物件。 若要這樣做，請開啟 Azure Powershell ISE。 複製並貼上下方的指令碼，接著使用您自己的值設定指令碼後執行。

        Get-AzureVM –ServiceName $serviceName –Name $vmName `
        | Remove-AzureAclConfig –EndpointName "web" `
        | Update-AzureVM

## <a name="next-steps"></a>後續步驟
[什麼是網路存取控制清單 (ACL)？](virtual-networks-acl.md)

