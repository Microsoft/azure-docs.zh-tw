---
title: 在 Azure 中重設 Windows VM 上的密碼或遠端桌面組態 | Microsoft Docs
description: 了解如何使用 Azure 入口網站或 Azure PowerShell，在使用傳統部署模型建立的 Windows VM 上重設帳戶密碼或遠端桌面服務。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 03/14/2017
ms.author: cynthn
ms.openlocfilehash: bbe8059b3a239570c2c9b25586dae9adbe25312d
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37931373"
---
# <a name="how-to-reset-the-remote-desktop-service-or-its-login-password-in-a-windows-vm-created-using-the-classic-deployment-model"></a>如何在使用傳統部署模型建立的 Windows VM 中重設遠端桌面服務或其登入密碼
> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [資源管理員和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 您也可以[針對使用 Resource Manager 部署模型建立的 VM 執行這些步驟](../reset-rdp.md)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]


如果您無法連線倒 Windows 虛擬機器 (VM)，您可以重設本機系統管理員密碼或重設「遠端桌面」服務組態。 您可以使用 Azure 入口網站或 Azure PowerShell 中的 VM 存取延伸模組來重設密碼。

## <a name="ways-to-reset-configuration-or-credentials"></a>重設組態或認證的方式
根據您的需求而定，您可以透過數種不同方式來重設遠端桌面服務和認證：

- [使用 Azure 入口網站重設](#azure-portal)
- [使用 Azure PowerShell 重設](#vmaccess-extension-and-powershell)

## <a name="azure-portal"></a>Azure 入口網站
您可以使用 [Azure 入口網站](https://portal.azure.com)來重設遠端桌面服務。 若要展開入口網站功能表，請按一下左上角的三個橫條，然後按一下 [虛擬機器 (傳統)]：

![瀏覽您的 Azure VM](./media/reset-rdp/Portal-Select-Classic-VM.png)

選取您的 Windows 虛擬機器，然後按一下 [重設遠端...]。以下對話方塊隨即出現，可供您重設「遠端桌面」組態：

![重設 RDP 組態頁面](./media/reset-rdp/Portal-RDP-Reset-Windows.png)

您也可以重設本機系統管理員帳戶的使用者名稱和密碼。 從您的 VM，按一下 [支援與疑難排解] > [重設密碼]。 將會顯示 [密碼重設] 刀鋒視窗︰

![密碼重設頁面](./media/reset-rdp/Portal-PW-Reset-Windows.png)

輸入新的使用者名稱和密碼，然後按一下 [儲存] 。

## <a name="vmaccess-extension-and-powershell"></a>VMAccess 延伸模組和 PowerShell
確認已在虛擬機器上安裝 VM 代理程式。 只要可以使用 VM 代理程式，您不需要先安裝 VMAccess 延伸模組即可使用。 使用以下命令確認已在虛擬機器上安裝 VM 代理程式。 (分別將 "myCloudService" 和 "myVM" 取代為雲端服務和 VM 的名稱。 您可以執行不搭配任何參數的 `Get-AzureVM` 來知道這些名稱)。

```powershell
$vm = Get-AzureVM -ServiceName "myCloudService" -Name "myVM"
write-host $vm.VM.ProvisionGuestAgent
```

如果 **write-host** 命令顯示 **True**，則會安裝 VM 代理程式。 如果顯示 **False**，請參閱 Azure 部落格文章 [VM 代理程式與延伸模組 - 第 2 部分](http://go.microsoft.com/fwlink/p/?linkid=403947&clcid=0x409) 中的指示和下載連結。

如果您使用入口網站建立虛擬機器，請檢查 `$vm.GetInstance().ProvisionGuestAgent` 是否會傳回 **True**。 否則，請使用此命令加以設定︰

```powershell
$vm.GetInstance().ProvisionGuestAgent = $true
```

此命令可避免在後續步驟中執行 **Set-AzureVMExtension** 命令時發生下列錯誤：「必須先在 VM 物件啟用佈建客體代理程式，才能設定 IaaS VM 存取延伸模組」。

### <a name="reset-the-local-administrator-account-password"></a>**重設本機系統管理員帳戶密碼**
使用目前的本機系統管理員帳戶名稱和新密碼建立登入認證，然後執行 `Set-AzureVMAccessExtension` ，如下所示。

```powershell
$cred=Get-Credential
Set-AzureVMAccessExtension –vm $vm -UserName $cred.GetNetworkCredential().Username `
    -Password $cred.GetNetworkCredential().Password  | Update-AzureVM
```

如果您輸入與目前帳戶不同的名稱，則 VMAccess 擴充功能會重新命名本機系統管理員帳戶、將密碼指派給該帳戶，以及發出遠端桌面登出。如果本機系統管理員帳戶已停用，則 VMAccess 延伸項目會將它啟用。

這些命令也會重設遠端桌面服務組態。

### <a name="reset-the-remote-desktop-service-configuration"></a>**重設遠端桌面服務組態**
若要重設遠端桌面服務組態，請執行下列命令：

```powershell
Set-AzureVMAccessExtension –vm $vm | Update-AzureVM
```

VMAccess 擴充功能會在虛擬機器上執行兩個命令：

```powershell
netsh advfirewall firewall set rule group="Remote Desktop" new enable=Yes
```

此命令會啟用允許連入遠端桌面流量 (使用 TCP 連接埠 3389) 的內建 Windows 防火牆群組。

```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
```

此命令會將 fDenyTSConnections 登錄值設為 0，以啟用遠端桌面連線。

## <a name="next-steps"></a>後續步驟
如果 Azure VM 存取延伸項目沒有回應，而且您無法重設密碼，可以[離線重設本機 Windows 密碼](../reset-local-password-without-agent.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 此方法是更進階的程序，會要求您將有問題的 VM 之中的虛擬硬碟連接至另一個 VM。 請先依照這篇文章中說明的步驟進行，並且只在最後無計可施時才嘗試離線密碼重設方法。

[Azure VM 延伸模組與功能](../extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

[透過 RDP 或 SSH 連接至 Azure 虛擬機器](http://msdn.microsoft.com/library/azure/dn535788.aspx)

[疑難排解以 Windows 為基礎之 Azure 虛擬機器的遠端桌面連線](../troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

