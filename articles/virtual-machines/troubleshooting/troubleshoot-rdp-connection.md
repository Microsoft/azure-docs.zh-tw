---
title: 無法透過 RDP 連接到 Azure 中的 Windows VM | Microsoft Docs
description: 針對無法在 Azure 中使用遠端桌面連接到 Windows 虛擬機器時的問題進行疑難排解
keywords: 遠端桌面錯誤、遠端桌面連線錯誤、無法連接到 VM、遠端桌面疑難排解
services: virtual-machines-windows
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: 0d740f8e-98b8-4e55-bb02-520f604f5b18
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 03/23/2018
ms.author: roiyz
ms.openlocfilehash: b73cdc7e55a60fbefc294a54f70daa9b2a11ced1
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452443"
---
# <a name="troubleshoot-remote-desktop-connections-to-an-azure-virtual-machine"></a>針對 Azure 虛擬機器的遠端桌面連線進行疑難排解
有各式各樣的原因可能導致 Windows 型 Azure 虛擬機器 (VM) 的遠端桌面通訊協定 (RDP) 連線失敗，讓您無法存取您的 VM。 問題可能與 VM 上的遠端桌面服務、網路連線或主機電腦上的遠端桌面用戶端有關。 本文將引導您完成一些可解決 RDP 連線問題的最常見方法。 

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。

<a id="quickfixrdp"></a>

## <a name="quick-troubleshooting-steps"></a>快速疑難排解步驟
在完成每個疑難排解步驟之後，請嘗試重新連接到 VM：

1. 重設遠端桌面組態。
2. 檢查網路安全性群組規則 / 雲端服務端點。
3. 檢閱 VM 主控台記錄檔。
4. 重設 VM 的 NIC。
5. 檢查 VM 資源健康狀態。
6. 重設您的 VM 密碼。
7. 重新啟動您的 VM。
8. 重新部署您的 VM。

如果您需要更詳細的步驟與說明，請繼續閱讀。 請確認區域網路設備 (例如路由器和防火牆) 沒有封鎖輸出 TCP 連接埠 3389，如[詳細的 RDP 疑難排解案例](detailed-troubleshoot-rdp.md)中所述。

> [!TIP]
> 如果 VM 的 [連接] 按鈕呈現灰色，而且您未透過 [Express Route](../../expressroute/expressroute-introduction.md) 或 [網站間 VPN](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) 連線來連接 Azure，您就必須先建立 VM 並為其指派公用 IP 位址，才能使用 RDP。 您可以深入了解 [Azure 中的公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)。


## <a name="ways-to-troubleshoot-rdp-issues"></a>針對 RDP 問題進行疑難排解的方法
您可以使用下列其中一種方法，針對使用 Resource Manager 部署模型建立的 VM 進行疑難排解：

* [Azure 入口網站](#using-the-azure-portal) - 適用於您需要快速重設 RDP 組態或使用者認證，而且未安裝 Azure 工具時。
* [Azure PowerShell](#using-azure-powershell) - 如果您已熟悉 PowerShell 提示字元，請使用 Azure PowerShell Cmdlet 快速重設 RDP 組態或使用者認證。

您也可以找到有關針對使用[傳統部署模型](#troubleshoot-vms-created-using-the-classic-deployment-model)建立的 VM 進行疑難排解的步驟。

<a id="fix-common-remote-desktop-errors"></a>

## <a name="troubleshoot-using-the-azure-portal"></a>使用 Azure 入口網站進行疑難排解
在每個疑難排解步驟完成之後，請再次嘗試連接到 VM。 如果仍然無法連線，請嘗試下一個步驟。

1. **重設您的 RDP 連線**。 當遠端連線已停用或 Windows 防火牆規則封鎖 RDP 時，此疑難排解步驟可重設 RDP 組態。
   
    在 Azure 入口網站中選取您的 VM。 向下捲動至 [設定] 窗格中接近清單底部的 [支援 + 疑難排解] 區段。 按一下 [重設密碼] 按鈕。 將 [模式] 設定為 [只重設組態]，然後按一下 [更新] 按鈕︰
   
    ![在 Azure 入口網站中重設 RDP 組態](./media/troubleshoot-rdp-connection/reset-rdp.png)
2. **確認網路安全性群組規則**。 使用 [IP 流量驗證](../../network-watcher/network-watcher-check-ip-flow-verify-portal.md)來確認網路安全性群組中的規則是否會封鎖虛擬機器的輸入或輸出流量。 您也可以檢閱有效的安全性群組規則，以確保輸入「允許」NSG 規則存在並已針對 RDP 連接埠 (預設值 3389) 設定優先順序。 如需詳細資訊，請參閱[使用有效安全性規則對 VM 流量流程進行疑難排解](../../virtual-network/diagnose-network-traffic-filter-problem.md)。

3. **檢閱 VM 開機診斷**。 此疑難排解步驟可檢閱 VM 主控台記錄檔，以判斷 VM 是否報告問題。 並非所有 VM 都已啟用開機診斷，所以此疑難排解步驟可能是選擇性的。
   
    特定疑難排解步驟已超出本文的範圍，但可能指出會影響 RDP 連線的更廣問題。 如需有關檢閱主控台記錄檔和 VM 螢幕擷取畫面的詳細資訊，請參閱 [VM 的開機診斷](boot-diagnostics.md)。

4. **重設 VM 的 NIC**。 如需詳細資訊，請參閱[如何重設 Azure Windows VM 的 NIC](../windows/reset-network-interface.md)。
5. **檢查 VM 資源健康狀態**。 此疑難排解步驟可確認 Azure 平台沒有任何可能影響 VM 連線的已知問題。
   
    在 Azure 入口網站中選取您的 VM。 向下捲動至 [設定] 窗格中接近清單底部的 [支援 + 疑難排解] 區段。 按一下 [資源健康狀態] 按鈕。 狀況良好的 VM 會報告為 [可用]：
   
    ![在 Azure 入口網站中檢查 VM 資源健康狀態](./media/troubleshoot-rdp-connection/check-resource-health.png)
6. **重設使用者認證**。 當您不確定或忘了認證時，此疑難排解步驟可重設本機系統管理員帳戶的密碼。  您登入 VM 後，應重設該使用者的密碼。
   
    在 Azure 入口網站中選取您的 VM。 向下捲動至 [設定] 窗格中接近清單底部的 [支援 + 疑難排解] 區段。 按一下 [重設密碼] 按鈕。 確定 [模式] 已設為 [重設密碼]，然後輸入您的使用者名稱和新密碼。 最後，按一下 [更新] 按鈕：
   
    ![在 Azure 入口網站中重設使用者認證](./media/troubleshoot-rdp-connection/reset-password.png)
7. **重新啟動您的 VM**。 此疑難排解步驟可以修正 VM 本身具有的任何基礎問題。
   
    在 Azure 入口網站中選取您的 VM，按一下 [概觀] 索引標籤。按一下 [重新啟動] 按鈕︰
   
    ![在 Azure 入口網站中重新啟動 VM](./media/troubleshoot-rdp-connection/restart-vm.png)
8. **重新部署您的 VM**。 此疑難排解步驟可將您的 VM 重新部署至 Azure 內的另一部主機，以更正任何基礎平台或網路問題。
   
    在 Azure 入口網站中選取您的 VM。 向下捲動至 [設定] 窗格中接近清單底部的 [支援 + 疑難排解] 區段。 按一下 [重新部署] 按鈕，然後再按一下 [重新部署]：
   
    ![在 Azure 入口網站中重新部署 VM](./media/troubleshoot-rdp-connection/redeploy-vm.png)
   
    此作業完成之後，會遺失暫時磁碟資料，並且會更新與 VM 相關聯的動態 IP 位址。

9. **確認路由**。 使用網路監看員的[下一個躍點](../../network-watcher/network-watcher-check-next-hop-portal.md)功能，確認路由不會防止流量從虛擬機器往返路由傳送。 您也可以檢閱有效路由，以查看網路介面的所有有效路由。 如需詳細資訊，請參閱[使用有效路由來針對 VM 流量流程進行疑難排解](../../virtual-network/diagnose-network-routing-problem.md)。

10. 確保任何內部部署防火牆或您電腦上的防火牆允許對 Azure 輸出 TCP 3389 流量。

如果您仍然遇到 RDP 問題，可以[開啟支援要求](https://azure.microsoft.com/support/options/)或閱讀[更詳細的 RDP 疑難排解概念和步驟](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

## <a name="troubleshoot-using-azure-powershell"></a>使用 Azure PowerShell 進行疑難排解
如果您尚未這樣做，請參閱 [安裝並設定最新的 Azure PowerShell](/powershell/azure/overview)。

下列範例使用 `myResourceGroup``myVM` 和 `myVMAccessExtension` 等變數。 以您自己的值取代這些變數名稱和位置。

> [!NOTE]
> 您可使用 [Set-AzureRmVMAccessExtension](/powershell/module/azurerm.compute/set-azurermvmaccessextension) PowerShell Cmdlet 重設使用者認證和 RDP 組態。 在下列範例中，`myVMAccessExtension` 是您指定為程序一部分的名稱。 如果您先前使用了 VMAccessAgent，則可以使用 `Get-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVM"` 來檢查 VM 的屬性，以取得現有擴充功能的名稱。 請查看輸出的 'Extensions' 區段底下來檢視名稱。

在每個疑難排解步驟完成之後，請再次嘗試連接到 VM。 如果仍然無法連線，請嘗試下一個步驟。

1. **重設您的 RDP 連線**。 當遠端連線已停用或 Windows 防火牆規則封鎖 RDP 時，此疑難排解步驟可重設 RDP 組態。
   
    下列範例會在位於 `WestUS` 且名為 `myResourceGroup` 的資源群組中名為 `myVM` 的 VM 上重設 RDP 連線：
   
    ```powershell
    Set-AzureRmVMAccessExtension -ResourceGroupName "myResourceGroup" `
        -VMName "myVM" -Location Westus -Name "myVMAccessExtension"
    ```
2. **確認網路安全性群組規則**。 此疑難排解步驟可確認您有可允許 RDP 流量的網路安全性群組規則。 RDP 的預設連接埠是 TCP 連接埠 3389。 當您建立您的 VM 時，可能不會自動建立可允許 RDP 流量的規則。
   
    首先，將網路安全性群組的所有組態資料指派給 `$rules` 變數。 下列範例會在名為 `myResourceGroup` 的資源群組中取得名為 `myNetworkSecurityGroup` 的網路安全性群組相關資訊：
   
    ```powershell
    $rules = Get-AzureRmNetworkSecurityGroup -ResourceGroupName "myResourceGroup" `
        -Name "myNetworkSecurityGroup"
    ```
   
    現在，檢視針對此網路安全性群組設定的規則。 確認存在一個規則可允許 TCP 連接埠 3389 進行輸入連線，如下所示︰
   
    ```powershell
    $rules.SecurityRules
    ```
   
    下列範例顯示可允許 RDP 流量的有效安全性規則。 您可以看到 `Protocol`、`DestinationPortRange`、`Access` 和 `Direction` 已正確設定︰
   
    ```powershell
    Name                     : default-allow-rdp
    Id                       : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/default-allow-rdp
    Etag                     : 
    ProvisioningState        : Succeeded
    Description              : 
    Protocol                 : TCP
    SourcePortRange          : *
    DestinationPortRange     : 3389
    SourceAddressPrefix      : *
    DestinationAddressPrefix : *
    Access                   : Allow
    Priority                 : 1000
    Direction                : Inbound
    ```
   
    如果您沒有可允許 RDP 流量的規則，請[建立網路安全性群組規則](../windows/nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 允許 TCP 連接埠 3389。
3. **重設使用者認證**。 當您不確定或忘了認證時，此疑難排解步驟可重設您指定之本機系統管理員帳戶的密碼。
   
    首先，藉由將認證指派給 `$cred` 變數，以指定使用者名稱和新密碼，如下所示︰
   
    ```powershell
    $cred=Get-Credential
    ```
   
    現在，更新 VM 上的認證。 下列範例會在位於 `WestUS` 且名為 `myResourceGroup` 的資源群組中名為 `myVM` 的 VM 上更新認證：
   
    ```powershell
    Set-AzureRmVMAccessExtension -ResourceGroupName "myResourceGroup" `
        -VMName "myVM" -Location WestUS -Name "myVMAccessExtension" `
        -UserName $cred.GetNetworkCredential().Username `
        -Password $cred.GetNetworkCredential().Password
    ```
4. **重新啟動您的 VM**。 此疑難排解步驟可以修正 VM 本身具有的任何基礎問題。
   
    下列範例會重新啟動名為 `myResourceGroup` 的資源群組中名為 `myVM` 的 VM：
   
    ```powershell
    Restart-AzureRmVM -ResourceGroup "myResourceGroup" -Name "myVM"
    ```
5. **重新部署您的 VM**。 此疑難排解步驟可將您的 VM 重新部署至 Azure 內的另一部主機，以更正任何基礎平台或網路問題。
   
    下列範例會重新部署位於 `WestUS` 且名為 `myResourceGroup` 的資源群組中名為 `myVM` 的 VM：
   
    ```powershell
    Set-AzureRmVM -Redeploy -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

6. **確認路由**。 使用網路監看員的[下一個躍點](../../network-watcher/network-watcher-check-next-hop-portal.md)功能，確認路由不會防止流量從虛擬機器往返路由傳送。 您也可以檢閱有效路由，以查看網路介面的所有有效路由。 如需詳細資訊，請參閱[使用有效路由來針對 VM 流量流程進行疑難排解](../../virtual-network/diagnose-network-routing-problem.md)。

7. 確保任何內部部署防火牆或您電腦上的防火牆允許對 Azure 輸出 TCP 3389 流量。

如果您仍然遇到 RDP 問題，可以[開啟支援要求](https://azure.microsoft.com/support/options/)或閱讀[更詳細的 RDP 疑難排解概念和步驟](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

## <a name="troubleshoot-vms-created-using-the-classic-deployment-model"></a>針對使用傳統部署模型建立的 VM 進行疑難排解
在每個疑難排解步驟完成之後，請嘗試重新連接到 VM。

1. **重設您的 RDP 連線**。 當遠端連線已停用或 Windows 防火牆規則封鎖 RDP 時，此疑難排解步驟可重設 RDP 組態。
   
    在 Azure 入口網站中選取您的 VM。 按一下 [...更多] 按鈕，然後按一下 [重設遠端存取]：
   
    ![在 Azure 入口網站中重設 RDP 組態](./media/troubleshoot-rdp-connection/classic-reset-rdp.png)
2. **確認雲端服務端點**。 此疑難排解步驟可確認您的雲端服務中有可允許 RDP 流量的端點。 RDP 的預設連接埠是 TCP 連接埠 3389。 當您建立您的 VM 時，可能不會自動建立可允許 RDP 流量的規則。
   
   在 Azure 入口網站中選取您的 VM。 按一下 [端點] 按鈕，以檢視目前為您的 VM 設定的端點。 確認存在可允許 TCP 連接埠 3389 上 RDP 流量的端點。
   
   下列範例顯示可允許 RDP 流量的有效端點：
   
   ![在 Azure 入口網站中確認雲端服務端點](./media/troubleshoot-rdp-connection/classic-verify-cloud-services-endpoints.png)
   
   如果您沒有可允許 RDP 流量的端點，請[建立雲端服務端點](../windows/classic/setup-endpoints.md)。 允許使用 TCP 連接至私人連接埠 3389。
3. **檢閱 VM 開機診斷**。 此疑難排解步驟可檢閱 VM 主控台記錄檔，以判斷 VM 是否報告問題。 並非所有 VM 都已啟用開機診斷，所以此疑難排解步驟可能是選擇性的。
   
    特定疑難排解步驟已超出本文的範圍，但可能指出會影響 RDP 連線的更廣問題。 如需有關檢閱主控台記錄檔和 VM 螢幕擷取畫面的詳細資訊，請參閱 [VM 的開機診斷](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/)。
4. **檢查 VM 資源健康狀態**。 此疑難排解步驟可確認 Azure 平台沒有任何可能影響 VM 連線的已知問題。
   
    在 Azure 入口網站中選取您的 VM。 向下捲動至 [設定] 窗格中接近清單底部的 [支援 + 疑難排解] 區段。 按一下 [資源健康狀態] 按鈕。 狀況良好的 VM 會報告為 [可用]：
   
    ![在 Azure 入口網站中檢查 VM 資源健康狀態](./media/troubleshoot-rdp-connection/classic-check-resource-health.png)
5. **重設使用者認證**。 當您不確定或忘了認證時，此疑難排解步驟可重設您指定之本機系統管理員帳戶的密碼。  您登入 VM 後，應重設該使用者的密碼。
   
    在 Azure 入口網站中選取您的 VM。 向下捲動至 [設定] 窗格中接近清單底部的 [支援 + 疑難排解] 區段。 按一下 [重設密碼] 按鈕。 輸入您的使用者名稱和新密碼。 最後，按一下 [儲存] 按鈕：
   
    ![在 Azure 入口網站中重設使用者認證](./media/troubleshoot-rdp-connection/classic-reset-password.png)
6. **重新啟動您的 VM**。 此疑難排解步驟可以修正 VM 本身具有的任何基礎問題。
   
    在 Azure 入口網站中選取您的 VM，按一下 [概觀] 索引標籤。按一下 [重新啟動] 按鈕︰
   
    ![在 Azure 入口網站中重新啟動 VM](./media/troubleshoot-rdp-connection/classic-restart-vm.png)

7. 確保任何內部部署防火牆或您電腦上的防火牆允許對 Azure 輸出 TCP 3389 流量。

如果您仍然遇到 RDP 問題，可以[開啟支援要求](https://azure.microsoft.com/support/options/)或閱讀[更詳細的 RDP 疑難排解概念和步驟](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

## <a name="troubleshoot-specific-rdp-errors"></a>針對特定 RDP 錯誤進行疑難排解
嘗試透過 RDP 連接到您的 VM 時，您可能會遇到特定的錯誤訊息。 以下是最常見的錯誤訊息：

* [遠端工作階段中斷，因為沒有提供授權的遠端桌面授權伺服器可以使用](troubleshoot-specific-rdp-errors.md#rdplicense)。
* [遠端桌面找不到電腦「名稱」](troubleshoot-specific-rdp-errors.md#rdpname)。
* [發生驗證錯誤。無法連絡本機安全性授權](troubleshoot-specific-rdp-errors.md#rdpauth)。
* [Windows 安全性錯誤：您的認證無法運作](troubleshoot-specific-rdp-errors.md#wincred)。
* [這部電腦無法連線到遠端電腦](troubleshoot-specific-rdp-errors.md#rdpconnect)。

## <a name="additional-resources"></a>其他資源
如果這些錯誤皆未發生，而您仍然無法透過「遠端桌面」連接到 VM，請閱讀詳細的 [遠端桌面疑難排解指南](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* 如需存取在 VM 上執行之應用程式的疑難排解步驟，請參閱[針對在 Azure VM 上執行之應用程式的存取進行疑難排解](../linux/troubleshoot-app-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
* 如果您在 Azure 中使用安全殼層 (SSH) 連線到 Linux VM 時遇到問題，請參閱[針對 Azure 中對 Linux VM 的 SSH 連線進行疑難排解](../linux/troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。


