---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 708098c7ed126705d7b8b561134e2bcf8c7f2fcd
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30197031"
---
您可以建立 VM 的遠端桌面連線，以連線至已部署至 VNet 的 VM。 一開始確認您可以連線至 VM 的最佳方法是使用其私人 IP 位址 (而不是電腦名稱) 進行連線。 這樣一來，您會測試以查看您是否可以連線，而不是否已正確設定名稱解析。

1. 找出私人 IP 位址。 在 Azure 入口網站中或使用 PowerShell 查看 VM 的屬性，即可找到 VM 的私人 IP 位址。

  - Azure 入口網站 - 在 Azure 入口網站中尋找您的虛擬機器。 檢視 VM 的屬性。 系統會列出私人 IP 位址。

  - PowerShell - 使用範例來檢視資源群組中的 VM 和私人 IP 位址清單。 使用此範例前，您不需要加以修改。

    ```powershell
    $VMs = Get-AzureRmVM
    $Nics = Get-AzureRmNetworkInterface | Where VirtualMachine -ne $null

    foreach($Nic in $Nics)
    {
      $VM = $VMs | Where-Object -Property Id -eq $Nic.VirtualMachine.Id
      $Prv = $Nic.IpConfigurations | Select-Object -ExpandProperty PrivateIpAddress
      $Alloc = $Nic.IpConfigurations | Select-Object -ExpandProperty PrivateIpAllocationMethod
      Write-Output "$($VM.Name): $Prv,$Alloc"
    }
    ```

2. 確認您已使用點對站 VPN 連線來連線至 VNet。
3. 在工作列上的搜尋方塊中輸入「RDP」或「遠端桌面連線」以開啟遠端桌面連線，然後選取 [遠端桌面連線]。 您也可以使用 PowerShell 中的 'mstsc' 命令開啟遠端桌面連線。 
4. 在 [遠端桌面連線] 中，輸入 VM 的私人 IP 位址。 您可以按一下 [顯示選項] 來調整其他設定，然後進行連線。

### <a name="to-troubleshoot-an-rdp-connection-to-a-vm"></a>針對 VM 的 RDP 連線進行疑難排解

如果您無法透過 VPN 連線與虛擬機器連線，請檢查下列各項：

- 確認您的 VPN 連線成功。
- 確認您是連線至 VM 的私人 IP 位址。
- 請使用 'ipconfig' 來檢查指派給所連線電腦上的乙太網路介面卡之 IPv4 位址。 如果 IP 位址位在您要連線的 VNet 位址範圍內，或在您 VPNClientAddressPool 的位址範圍內，這稱為重疊位址空間。 當您的位址空間以這種方式重疊時，網路流量不會連線到 Azure，它會保留在本機網路上。
- 如果您可以使用私人 IP 位址 (而非電腦名稱) 來連線至 VM，請確認您已正確設定 DNS。 如需 VM 的名稱解析運作方式的詳細資訊，請參閱 [VM 的名稱解析](../articles/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。
- 請確認 VPN 用戶端設定套件是在針對 VNet 指定的 DNS 伺服器 IP 位址之後產生。 如果您已更新 DNS 伺服器 IP 位址，請產生並安裝新的 VPN 用戶端設定套件。
- 如需 RDP 連線的詳細資訊，請參閱[針對 VM 的遠端桌面連線進行疑難排解](../articles/virtual-machines/windows/troubleshoot-rdp-connection.md)。