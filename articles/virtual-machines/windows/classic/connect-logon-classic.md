---
title: 登入傳統 Azure VM | Microsoft Docs
description: 使用 Azure 入口網站來登入以傳統部署模型建立的 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: 3c1239ed-07dc-48b8-8b3d-dc8c5f0ab20e
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/30/2017
ms.author: cynthn
ms.openlocfilehash: 87ecc65d2d4802ae826f3260b66b26e0bbe414e6
ms.sourcegitcommit: 909469bf17211be40ea24a981c3e0331ea182996
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "34012354"
---
# <a name="log-on-to-a-windows-virtual-machine-using-the-azure-portal"></a>使用 Azure 入口網站登入 Windows 虛擬機器
在 Azure 入口網站中，使用 [連接]  按鈕來啟動遠端桌面工作階段，並登入 Windows VM。

您想要連線至 Linux VM 嗎？ 請參閱[如何登入執行 Linux 的虛擬機器](../../linux/mac-create-ssh-keys.md)。

<!--
Deleting, but not 100% sure
Learn how to [perform these steps using new Azure portal](../connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
-->

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如需如何使用 Resource Manager 模型登入 VM 的詳細資訊，請參閱[這裡](../connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

## <a name="connect-to-the-virtual-machine"></a>連接至虛擬機器
1. 登入 Azure 入口網站。
2. 按一下您想要存取的虛擬機器。 名稱會在 [所有資源] 窗格中列出。

    ![Virtual-machine-locations](./media/connect-logon/azureportaldashboard.png)

1. 按一下虛擬機器屬性頁面上的 [連線] 按鈕。 
2. 在 [連線至虛擬機器] 頁面上，維持選取適當的選項，然後按一下 [下載 RDP 檔案]。
2. 開啟下載的 RDP 檔案，然後在出現提示時按一下 [連線]。 
2. 您會收到警告，表示 `.rdp` 來自未知的發行者。 這是正常現象。 在 [遠端桌面] 視窗中按一下 [連接]  以繼續。
   
    ![未知發行者相關警告的螢幕擷取畫面。](./media/connect-logon/rdp-warn.png)
3. 在 [Windows 安全性] 視窗中，選取 [更多選擇]，然後選取 [使用不同的帳戶]。 輸入虛擬機器上帳戶的認證，然後按一下 [確定]。
   
     **本機帳戶** - 這通常是您在建立虛擬機器時所指定的本機帳戶使用者名稱和密碼。 在此案例中，虛擬機器的名稱是網域，而它的輸入格式為 *vmname*&#92;*username*。  
   
    **已加入網域的 VM** - 如果 VM 屬於網域，請輸入 *Domain*&#92;*Username* 格式的使用者名稱。 此帳戶也必須屬於系統管理員群組，或者已授與遠端存取 VM 的權限。
   
    **網域控制站** - 如果 VM 是網域控制站，請輸入該網域的網域系統管理員帳戶的使用者名稱和密碼。
4. 按一下 [是]  來確認虛擬機器的身分識別，並完成登入。
   
   ![顯示驗證 VM 身分識別相關訊息的螢幕擷取畫面。](./media/connect-logon/cert-warning.png)

## <a name="next-steps"></a>後續步驟
* 如果 [連接] 按鈕無法使用，或是有其他「遠端桌面」連線問題，請嘗試重設組態。 從虛擬機器儀表板按一下 [重設遠端存取]。

    ![Reset-remote-access](./media/connect-logon/virtualmachine_dashboard_reset_remote_access.png)

* 如果是您的密碼有問題，請嘗試重設密碼。 按一下沿著虛擬機器儀表板左側邊緣 [支援與疑難排解] 底下的 [重設密碼]。

    ![Reset-password](./media/connect-logon/virtualmachine_dashboard_reset_password.png)

如果這些秘訣無效，或者不是您所需要的，請參閱[針對以 Windows 為基礎之 Azure 虛擬機器的遠端桌面連線進行疑難排解](../troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 本文會逐步帶領您診斷及解決常見的問題。
