---
title: 如何針對 VHD 建立常見問題進行疑難排解 | Microsoft Docs
description: VHD 建立常見疑難排解問題的解答。
services: Azure Marketplace
documentationcenter: ''
author: HannibalSII
manager: ''
editor: ''
ms.assetid: e39563d8-8646-4cb7-b078-8b10ac35b494
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Azure
ms.workload: na
ms.date: 09/26/2016
ms.author: hascipio; v-divte
ms.openlocfilehash: c4e88a9fbb15dd90d619b159ae1065dfacc1907f
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39713392"
---
# <a name="how-to-troubleshoot-common-issues-encountered-during-vhd-creation"></a>如何針對 VHD 建立常見問題進行疑難排解
這篇文章可協助 Azure Marketplace 發行者和 (或) 共同管理員解決在發行或管理其虛擬機器解決方案時發生的問題或常見問題。

1. 如何變更主機的名稱？
   
    建立 VM 之後，使用者就無法更新主機的名稱。
2. 如何重設遠端桌面服務或其登入密碼？
   
   * [Windows VM 的參考](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-reset-rdp/)
   * [Linux VM 的參考](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-classic-reset-access/)
3. 如何產生新 ssh 憑證？
   
   請參閱此連結：[https://azure.microsoft.com/documentation/articles/virtual-machines-linux-classic-reset-access/](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-classic-reset-access/)
4. 如何設定開放式 VPN 憑證？
   
   請參閱此連結：[https://azure.microsoft.com/documentation/articles/vpn-gateway-point-to-site-create/](https://azure.microsoft.com/documentation/articles/vpn-gateway-point-to-site-create/)
5. 在 Microsoft Azure 虛擬機器環境中執行 Microsoft 伺服器軟體的支援原則為何 (infrastructure-as-a-service)？
   
   請參閱此連結：[https://support.microsoft.com/kb/2721672](https://support.microsoft.com/kb/2721672)
6. 虛擬機器是否有任何唯一識別碼？
   
   Azure 會編碼每個 VM 中的 Azure VM 唯一識別碼。 請參閱這個部落格和文件中的詳細資料。
7. 在 VM 中，如何在啟動工作中管理自訂指令碼延伸模組？
   
   請參閱此連結：[https://azure.microsoft.com/documentation/articles/virtual-machines-windows-extensions-customscript/](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-extensions-customscript/)
8. 如何使用上傳至進階儲存體的 VHD 從 Azure 入口網站建立 VM？
   
   我們尚未支援這項功能。
9. Azure Marketplace 中是否支援 32 位元應用程式？
   
   如需支援原則的詳細資料，請參閱此連結︰[https://support.microsoft.com/kb/2721672](https://support.microsoft.com/kb/2721672)
10. 每次嘗試從我的 VHD 建立映像時都會在 PowerShell 中收到錯誤「.VHD 已經在映像儲存機制上註冊為資源」。 我之前未建立任何映像，也在 Azure 中找不到任何具有這個名稱的映像。 如何解決這個問題？
    
    如果使用者已從這個 VHD 佈建 VM，並且鎖定該 VHD，則通常會發生這種情況。 請確認未從這個 VHD 配置任何 VM。 如果錯誤仍然存在，則請使用這個連結或從發佈入口網站提出這個錯誤的支援票證 (第 11 個問題的答案會提供詳細資料)。