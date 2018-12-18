---
title: 連接到 Windows Server VM | Microsoft Docs
description: 了解如何使用 Azure 入口網站和 Resource Manager 部署模型來連接和登入 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ef62b02e-bf35-468d-b4c3-71b63fe7f409
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 09/13/2018
ms.author: cynthn
ms.openlocfilehash: b9cce5658b705e9d3255d2662b2a0157a2e548c3
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409023"
---
# <a name="how-to-connect-and-log-on-to-an-azure-virtual-machine-running-windows"></a>如何連接和登入執行 Windows 的 Azure 虛擬機器
您會使用 Azure 入口網站中的 [連線] 按鈕，從 Windows 桌面啟動遠端桌面 (RDP) 工作階段。 首先您必須連線到虛擬機器，然後登入。

若要從 Mac 連線到 Windows VM，您需要安裝適用於 Mac 的 RDP 用戶端，例如 [Microsoft 遠端桌面](https://itunes.apple.com/app/microsoft-remote-desktop/id715768417)。

## <a name="connect-to-the-virtual-machine"></a>連接到虛擬機器
1. 如果您尚未登入 [Azure 入口網站](https://portal.azure.com/)，請先登入。
2. 在左功能表上，選取 [虛擬機器]。
3. 從清單中選取虛擬機器。
4. 在虛擬機器頁面上方，選取 [連線]。
2. 在 [連線至虛擬機器] 頁面上，選取適當的選項，然後選取 [下載 RDP 檔案]。
2. 開啟下載的 RDP 檔案，然後在出現提示時選取 [連線]。 
2. 您會收到警告，表示 .rdp 檔案來自未知的發行者。 這是預期行為。 在 [遠端桌面連線] 視窗中，選取 [連線] 以繼續。
   
    ![未知發行者相關警告的螢幕擷取畫面。](./media/connect-logon/rdp-warn.png)
3. 在 [Windows 安全性] 視窗中，選取 [其他選擇]，然後選取 [使用不同的帳戶]。 輸入虛擬機器上帳戶的認證，然後選取 [確定]。
   
     **本機帳戶**：這通常是您在建立虛擬機器時所指定的本機帳戶使用者名稱與密碼。 在此案例中，虛擬機器的名稱是網域，而它的輸入格式為 *vmname*&#92;*username*。  
   
    **已加入網域的 VM**：如果 VM 屬於某個網域，請以 *Domain*&#92;*Username* 格式輸入使用者名稱。 此帳戶也必須屬於系統管理員群組，或者已被授與遠端存取 VM 的權限。
   
    **網域控制站**：如果 VM 是網域控制站，請輸入該網域的網域系統管理員帳戶的使用者名稱與密碼。
4. 選取 [是] 以確認虛擬機器的身分識別，並完成登入。
   
   ![顯示驗證 VM 身分識別相關訊息的螢幕擷取畫面。](./media/connect-logon/cert-warning.png)


   > [!TIP]
   > 如果入口網站中的 [連線] 按鈕呈現灰色，而且您未透過 [Express Route](../../expressroute/expressroute-introduction.md) 或[站台對站台 VPN](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) 連線來連線到 Azure，您必須先建立 VM 並為它指派公用 IP 位址，才能使用 RDP。 如需詳細資訊，請參閱 [Azure 中的公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)。
   > 
   > 


## <a name="next-steps"></a>後續步驟
如果您無法連線，請參閱[針對遠端桌面連線進行疑難排解](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 

