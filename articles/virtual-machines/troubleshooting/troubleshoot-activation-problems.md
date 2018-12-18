---
title: 針對 Azure 中的 Windows 虛擬機器啟用問題進行疑難排解 | Microsoft Docs
description: 針對 Azure 中的 Windows 虛擬機器啟用問題提供疑難排解步驟
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: genlin
manager: willchen
editor: ''
tags: top-support-issue, azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: e8ecdf1fffb51c0b8e9ce996307595a5444a64ee
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411756"
---
# <a name="troubleshoot-azure-windows-virtual-machine-activation-problems"></a>針對 Azure Windows 虛擬機器啟用問題進行疑難排解

如果您在啟用從自訂映像建立的 Azure Windows 虛擬機器 (VM) 時遇到問題，您可以使用本文件中提供的資訊來針對問題進行疑難排解。 

## <a name="understanding-azure-kms-endpoints-for-windows-product-activation-of-azure-virtual-machines"></a>了解 Azure 虛擬機器的 Windows 產品啟用適用的 Azure KMS 端點
Azure 會根據 VM 所在的雲端區域使用不同的端點來啟用 KMS。 使用此疑難排解指南時，請使用適用於您的區域的適當 KMS 端點。

* Azure 公用雲端區域： kms.core.windows.net:1688
* Azure 中國國家雲端區域： kms.core.chinacloudapi.cn:1688
* Azure 德國國家雲端區域： kms.core.chinacloudapi.cn:1688
* Azure 美國政府國家雲端區域： kms.core.chinacloudapi.cn:1688

## <a name="symptom"></a>徵狀

當您嘗試啟用 Azure Windows VM 時，收到類似下列範例的錯誤訊息：

**錯誤：0xC004F074。軟體授權服務報告指出無法啟用電腦。無法聯繫金鑰管理服務 (KMS)。如需其他資訊，請參閱應用程式事件記錄檔。**

## <a name="cause"></a>原因
一般而言，如果不是使用正確的 KMS 用戶端安裝識別碼來設定 Windows VM，或 Windows VM 無法連線到 Azure KMS 服務 (kms.core.windows.net，連接埠 1668)，就會發生 Azure VM 啟用問題。 

## <a name="solution"></a>解決方法

>[!NOTE]
>如果您使用站對站 VPN 和強制通道，請參閱[使用 Azure 自訂路由來啟用搭配強制通道的 KMS 啟用](http://blogs.msdn.com/b/mast/archive/2015/05/20/use-azure-custom-routes-to-enable-kms-activation-with-forced-tunneling.aspx) (英文)。 
>
>如果您使用 ExpressRoute 並有已發佈的預設路由，請參閱 [Azure VM 可能無法透過 ExpressRoute 進行啟用](http://blogs.msdn.com/b/mast/archive/2015/12/01/azure-vm-may-fail-to-activate-over-expressroute.aspx) (英文)。

### <a name="step-1-configure-the-appropriate-kms-client-setup-key-for-windows-server-2016-and-windows-server-2012-r2"></a>步驟 1：設定適當的 KMS 用戶端安裝識別碼 (適用於 Windows Server 2016 和 Windows Server 2012 R2)

針對從 Windows Server 2016 或 Windows Server 2012 R2 的自訂映像建立的 VM，您必須為 VM 設定適當的 KMS 用戶端安裝識別碼。

此步驟不適用於 Windows 2012 或 Windows 2008 R2。 它使用只有 Windows Server 2016 和 Windows Server 2012 R2 才支援的「自動化虛擬機器啟用」(AVMA) 功能。

1. 在提升權限的命令提示字元執行 **slmgr.vbs /dlv**。 檢查輸出中的描述值，然後判斷它是建立自零售 (RETAIL通路) 還是大量授權媒體 (VOLUME_KMSCLIENT)：
  
    ```
    cscript c:\windows\system32\slmgr.vbs /dlv
    ```

2. 如果 **slmgr.vbs /dlv** 顯示 RETAIL 通路，請執行下列命令來設定所使用之 Windows Server 版本的 [KMS 用戶端安裝識別碼](https://technet.microsoft.com/library/jj612867%28v=ws.11%29.aspx?f=255&MSPPError=-2147217396)，然後強制它重試啟用： 

    ```
    cscript c:\windows\system32\slmgr.vbs /ipk <KMS client setup key>

    cscript c:\windows\system32\slmgr.vbs /ato
     ```

    例如，針對 Windows Server 2016 Datacenter，您會執行下列命令：

    ```
    cscript c:\windows\system32\slmgr.vbs /ipk CB7KF-BWN84-R7R2Y-793K2-8XDDG
    ```

### <a name="step-2-verify-the-connectivity-between-the-vm-and-azure-kms-service"></a>步驟 2：確認 VM 與 Azure KMS 服務之間的連線

1. 下載 [PSping](http:/technet.microsoft.com/sysinternals/jj729731.aspx) 工具，並將它解壓縮至未啟用 VM 中的本機資料夾。 

2. 移至 [開始]，搜尋 Windows PowerShell，在 [Windows PowerShell] 上按一下滑鼠右鍵，然後選取 [以系統管理員身分執行]。

3. 確定已將 VM 設定為使用正確的 Azure KMS 伺服器。 若要這樣做，請執行下列命令：
  
    ```
    iex “$env:windir\system32\cscript.exe $env:windir\system32\slmgr.vbs /skms
    kms.core.windows.net:1688
    ```
    此命令應該會傳回：金鑰管理服務電腦名稱已成功設定為 kms.core.windows.net:1688。

4. 使用能夠連線到 KMS 伺服器的 Psping 來進行確認。 切換到您將所下載的 Pstools.zip 解壓縮的資料夾，然後執行下列命令：
  
    ```
    \psping.exe kms.core.windows.net:1688
    ```
  
  在輸出的倒數第二行，確定您看到的是：Sent = 4, Received = 4, Lost = 0 (0% loss)。

  如果 Lost 大於 0 (零)，即表示 VM 無法連線到 KMS 伺服器。 在此情況下，如果 VM 位於虛擬網路中並已指定自訂的 DNS 伺服器，您就必須確定 DNS 伺服器能夠解析 kms.core.windows.net。 或者，將 DNS 伺服器變更為能夠解析 kms.core.windows.net 的伺服器。

  請注意，如果您從虛擬網路中移除所有 DNS 伺服器，VM 將會使用 Azure 的內部 DNS 服務。 此服務可以解析 kms.core.windows.net。
  
此外，請確認來賓防火牆未設定成會封鎖啟用嘗試。

5. 在您確認可成功連線到 kms.core.windows.net 之後，請在該提升權限的 Windows PowerShell 提示中執行下列命令。 此命令會多次嘗試啟用。

    ```
    1..12 | % { iex “$env:windir\system32\cscript.exe $env:windir\system32\slmgr.vbs /ato” ; start-sleep 5 }
    ```

成功啟用時，會傳回類似以下的資訊：

**正在啟用 Windows(R) Server Datacenter 版本 (12345678-1234-1234-1234-12345678) … 產品已成功啟用。**

## <a name="faq"></a>常見問題集 

### <a name="i-created-the-windows-server-2016-from-azure-marketplace-do-i-need-to-configure-kms-key-for-activating-the-windows-server-2016"></a>我從 Azure Marketplace 建立了 Windows Server 2016。 我是否需要設定 KMS 金鑰來啟用 Windows Server 2016？ 
 
否。 Azure Marketplace 中的映像已設定適當的 KMS 用戶端安裝識別碼。 

### <a name="does-windows-activation-work-the-same-way-regardless-if-the-vm-is-using-azure-hybrid-use-benefit-hub-or-not"></a>是否不論 VM 是否使用 Azure Hybrid Use Benefit (HUB)，Windows 啟用的運作方式都相同？ 
 
是。 
 
### <a name="what-happens-if-windows-activation-period-expires"></a>如果 Windows 啟用期間已到期，會發生什麼情況？ 
 
當寬限期已到期而 Windows 仍然未啟用時，Windows Server 2008 R2 和更新版的 Windows 將會顯示與啟用相關的額外通知。 桌面桌布會維持黑色，而 Windows Update 將只會安裝安全性和重大更新，而不會安裝選用更新。 請參閱[授權條件](http://technet.microsoft.com/library/ff793403.aspx)頁面底部的＜通知＞一節。   

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。
如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
