---
title: 在 Azure 資訊安全中心套用系統更新 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議的「套用系統更新」和「在系統更新之後重新開機」。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: e5bd7f55-38fd-4ebb-84ab-32bd60e9fa7a
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/11/2017
ms.author: terrylan
ms.openlocfilehash: 9f7924f3f0975dc32fdf5b8e1b89a1fb8e9b7d57
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23040483"
---
# <a name="apply-system-updates-in-azure-security-center"></a>在 Azure 資訊安全中心套用系統更新
Azure 資訊安全中心每日監視 Windows 和 Linux 虛擬機器 (VM) 和電腦是否有遺漏的作業系統更新。 資訊安全中心會根據 Windows 電腦上設定的服務，從 Windows Update 或 Windows Server Update Services (WSUS) 擷取可用的安全性和重大更新清單。 資訊安全中心也會檢查 Linux 系統中的最新更新。 如果您的 VM 或電腦遺漏系統更新，資訊安全中心會建議您套用系統更新。

## <a name="implement-the-recommendation"></a>實作建議
套用系統更新會以建議的形式在資訊安全中心中呈現。 如果您的 VM 或電腦遺漏系統更新，這項建議會出現在 [建議] 和 [計算] 下方。  選取建議即會開啟 [套用系統更新] 儀表板。

在此範例中，我們使用 [計算]。

1. 在 [資訊安全中心] 主功能表下選取 [計算]。

   ![選取 [計算]][1]

2. 在 [計算] 下方，選取 [遺漏系統更新]。 [套用系統更新] 儀表板隨即開啟。

   ![[套用系統更新] 儀表板][2]

   儀表板的頂端提供：

    - 遺漏系統更新的 Windows 和 Linux VM 以及電腦總數。
    - VM 和電腦遺漏的重大更新總數。
    - VM 和電腦遺漏的安全性更新總數。

  儀表板底部會列出 VM 和電腦所有遺漏的更新，以及缺少更新的嚴重性。  此清單包括：

    - 名稱：遺漏更新的名稱。
    - 否。 和電腦數目：遺漏此更新的 VM 和電腦總數。
    - 狀態：建議的目前狀態：

      - 未處理：建議尚未處理。
      - 進行中：正在將建議套用到資源，且您不需要採取任何動作。
      - 已解決︰建議動作已完成。 (若問題已解決，該項目會呈現暗灰色)。

    - 嚴重性：描述該特定建議的嚴重性：

      - 高：某個有意義的資源 (應用程式、虛擬機器或網路安全性群組) 有弱點存在，並且需要注意。
      - 中：需要執行非關鍵性步驟或其他步驟才能完成程序或消除弱點。
      - 低：應該處理但不需要立即注意的弱點。 (預設不會顯示嚴重性低的建議，但是如果您要檢視它們，則可以篩選嚴重性低的建議)。

3. 選取清單中遺漏的更新以檢視詳細資料。

   ![遺漏的安全性更新][3]

4. 選取頂端功能區中的 [搜尋] 圖示。  Log Analytics 搜尋查詢隨即開啟，篩選條件會設為遺漏更新的電腦。

   ![Log Analytics 搜尋][4]

5. 從清單中選取電腦以了解更多資訊。 會出現僅針對該電腦篩選出的其他搜尋結果。

    ![Log Analytics 搜尋][5]

## <a name="reboot-after-system-updates"></a>在系統更新之後重新開機
1. 返回 [建議]  刀鋒視窗。 在您套用系統更新之後會產生新的項目，稱為「在系統更新之後重新開機」。 此項目可讓您知道您需要重新啟動 VM 以完成套用系統更新的程序。

   ![在系統更新之後重新開機][6]
2. 選取 [在系統更新之後重新開機] 。 這會開啟 [重新啟動正在等待以完成系統更新]  刀鋒視窗，其中顯示 VM，您必須加以重新啟動才能完成套用系統清單更新的程序。

   ![重新啟動擱置中][7]

從 Azure 重新啟動 VM 以完成程序。

## <a name="next-steps"></a>後續步驟
如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) -- 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 尋找有關 Azure 安全性與相容性的部落格文章。

<!--Image references-->
[1]: ./media/security-center-apply-system-updates/missing-system-updates.png
[2]:./media/security-center-apply-system-updates/apply-system-updates.png
[3]: ./media/security-center-apply-system-updates/detail-on-missing-update.png
[4]: ./media/security-center-apply-system-updates/log-search.png
[5]: ./media/security-center-apply-system-updates/search-details.png
[6]: ./media/security-center-apply-system-updates/reboot-after-system-updates.png
[7]: ./media/security-center-apply-system-updates/restart-pending.png
