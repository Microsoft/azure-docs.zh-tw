---
title: 使用 Azure 資訊安全中心建議增強安全性 | Microsoft Docs
description: " 了解如何使用「Azure 資訊安全中心」中的安全性原則和建議，來協助降低安全性攻擊的危害。 "
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: ''
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/18/2017
ms.author: terrylan
ms.openlocfilehash: 0616f5e501324bfd821c1455ce234602f1fcf1bd
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23040973"
---
# <a name="use-azure-security-center-recommendations-to-enhance-security"></a>使用 Azure 資訊安全中心建議增強安全性
您可以設定安全性原則，然後實作 Azure 資訊安全中心提供的建議，以降低發生重大安全性事件的機會。 本文說明如何使用資訊安全中心的安全性原則和建議，以協助減少安全性攻擊。

> [!NOTE]
> 本文是根據資訊安全中心[規劃與操作指南](security-center-planning-and-operations-guide.md)中所介紹的角色和概念。 最好先檢閱規劃指南再繼續。
>
>

## <a name="managing-security-recommendations"></a>管理安全性建議
安全性原則定義了針對指定之訂用帳戶或資源群組內的資源所建議的一組控制項。 在資訊安全中心，您可以根據公司的安全性需求來定義原則。 若要深入了解，請參閱[在資訊安全中心設定安全性原則](security-center-policies.md)。

資源群組的安全性原則繼承自訂用帳戶層級。

![安全性原則繼承][1]

如果您在特定資源群組中需要自訂原則，您可以在此資源群組中停用繼承。 若要停用，請在 [安全性原則] 刀鋒視窗上，將 [繼承] 設定為 [唯一]，並自訂資訊安全中心已顯示建議的控制。

比方說，如果您的工作負載不需要 SQL Database 透明資料加密 (TDE) 原則，請在訂用帳戶層級關閉此原則，只在需要 SQL TDE 的資源群組中啟用。

> [!NOTE]
> 如果訂用帳戶層級原則與資源群組層級原則間有衝突，將會優先採用資源群組層級原則。
>
>

資訊安全中心會分析 Azure 資源的安全性狀態。 當資訊安全中心發現潛在的安全性弱點時，它會根據安全性原則中設定的控制來提出建議。 這些建議會引導您完成設定所需安全性控制的程序。

資訊安全中心目前的原則建議著重於系統更新、作業系統設定、子網路和虛擬機器 (VM) 上的網路安全性群組、SQL Database 稽核、SQL Database TDE 和 Web 應用程式防火牆。 有關資訊安全中心建議的最新消息，請參閱[在資訊安全中心管理安全性建議](security-center-recommendations.md)。

## <a name="scenario"></a>案例
此案例示範如何使用資訊安全中心，透過監控資訊安全中心建議並採取動作，以協助降低發生重大安全性事件的機會。 此案例使用虛構公司 Contoso，以及資訊安全中心[規劃與操作指南](security-center-planning-and-operations-guide.md#security-roles-and-access-controls)中出現的角色。 這些角色代表一些個人和團隊，他們可能使用資訊安全中心來執行不同的安全性相關工作。 角色如下：

![案例角色][2]

Contoso 最近將一些內部部署資源移轉至 Azure。 Contoso 想要實作和維護防護措施，以減少弱點來避免雲端中的資源遭受安全性攻擊。

## <a name="recommended-solution"></a>建議的解決方案
解決方案是使用資訊安全中心來防止和偵測安全性弱點。 Contoso 可以透過 Azure 訂用帳戶存取資訊安全中心。 所有 Azure 訂用帳戶會自動啟用資訊安全中心的[免費層](security-center-pricing.md)，而訂用帳戶中的所有 VM 上會啟用資料收集。

任職於 Contoso IT 安全部門的 David 使用資訊安全中心設定**安全性原則**。 資訊安全中心會分析 Contoso Azure 資源的安全性狀態。 當資訊安全中心發現潛在的安全性弱點時，它會根據安全性原則中設定的控制來提出**建議**。

雲端工作負載擁有者 Jeff 負責根據 Contoso 的安全性原則，實作和維護防護措施。 Jeff 可以監視資訊安全中心所建立的建議，以套用防護措施。 這些建議會引導 Jeff 完成設定所需安全性控制的程序。

為了讓 Jeff 實作和維護防護措施並消除安全性弱點，他必須︰

- 監視資訊安全中心提供的安全性建議
- 評估安全性建議，並決定是否應該套用或解除
- 套用安全性建議

讓我們循著 Jeff 的步驟，以了解他如何使用資訊安全中心的建議，在過程中經由引導來設定控制，以消除安全性弱點。

## <a name="how-to-implement-this-solution"></a>如何實作此解決方案
Jeff 登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)，並開啟資訊安全中心主控台。 在每日的監視活動中，他會執行下列步驟來檢查是否有安全性建議︰

1. Jeff 選取 [建議] 圖格以開啟 [建議]。
   ![選取建議圖格][3]
2. Jeff 檢閱建議清單。 他看到資訊安全中心已依優先順序提供建議清單 (優先順序由高至低)。 他決定採用清單上的「高」優先順序建議。 他選取 [建議] 下方的 [安裝 Endpoint Protection]。
3. [安裝 Endpoint Protection] 隨即開啟，顯示未啟用反惡意程式碼的 VM 清單。 Jeff 檢閱 VM 清單，選取所有 VM，然後選取 [安裝在 3 個 VM 上]。
   ![安裝端點保護][4]
4. [選取 Endpoint Protection] 隨時開啟，提供兩個反惡意程式碼解決方案給 Jeff。 Jeff 選取 [Microsoft Antimalware] 解決方案。
5. 將會顯示反惡意程式碼解決方案的其他相關資訊。 Jeff 選取 [建立]。
   ![Microsoft antimalware][5]
6. Jeff 在 [安裝] 下方輸入必要的組態設定，然後選取 [確定]。

[Microsoft Antimalware](../security/azure-security-antimalware.md)現在已在選取的 VM 上使用。

Jeff 繼續瀏覽高優先權和中優先順序的建議，然後決定實作。 Jeff 參考[管理安全性建議](security-center-recommendations.md)一文，以了解建議及每一項建議在套用時會怎麼做。

Jeff 了解 [Microsoft Security Response Center (MSRC)](../security/azure-security-response-center.md) 會執行 Azure 網路和基礎結構的獨特安全性監視，並接收來自協力廠商的威脅情報和濫用客訴。 如果 Jeff 提供 Contoso Azure 訂用帳戶的安全性連絡人詳細資料，當 MSRC 發現有非法或未經授權的一方存取 Contoso 的客戶資料時，Microsoft 會連絡 Contoso。 讓我們追蹤 Jeff 如何套用**提供安全性連絡人詳細資料**建議 (上述建議清單中具有「中」嚴重性的建議)。

1. Jeff 選取 [建議] 下方的 [提供安全性連絡人詳細資料]，以開啟 [提供安全性連絡人詳細資料]。
2. Jeff 選取要提供連絡人資訊的 Azure 訂用帳戶。 第二個 [提供安全性連絡人詳細資料]  刀鋒視窗隨即開啟。
   ![安全性連絡人詳細資料][6]
3. 在 [提供安全性連絡人詳細資料] 下方，Jeff 輸入：

  - 安全性連絡人電子郵件地址，以逗號分隔 (他可以輸入的電子郵件地址數目沒有限制)
  - 一個安全性連絡人電話號碼

4. Jeff 也開啟 [傳送給我有關警示的電子郵件] 選項，以接收有關高嚴重性警示的電子郵件。
5. Jeff 選取 [確定]，將安全性連絡人資訊套用至 Contoso 的訂用帳戶。

最後，Jeff 檢閱低優先順序的建議 [修復 OS 弱點]，認為這項建議不適用。 他想要解除這項建議。 Jeff 選取右邊出現的三個點，然後選取 [解除]。
   ![解除建議][7]

## <a name="conclusion"></a>結論
監控資訊安全中心的建議可能有助於在發生攻擊之前消除安全性弱點。 您可以使用資訊安全中心的安全性原則來實作和維護防護措施，以防止安全性事件。

## <a name="next-steps"></a>後續步驟
本案例示範如何使用資訊安全中心的安全性原則和建議，以協助減少安全性攻擊。 請參閱[事件回應案例](security-center-incident-response.md)，以了解如何在攻擊發生前備妥事件回應計劃。

若要深入了解資訊安全中心，請參閱：

* [安全性健康情況監視](security-center-monitoring.md) — 了解如何監視 Azure 資源的健康清況。
* [管理與回應安全性警示](security-center-managing-and-responding-alerts.md) — 了解如何管理與回應安全性警示。
* [監視和處理安全性事件](security-center-events-dashboard.md)-了解如何監視和處理一段時間下來收集到的安全性事件。
* [監視合作夥伴解決方案](security-center-partner-solutions.md) — 了解如何監視合作夥伴解決方案的健全狀態。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 取得最新的 Azure 安全性新聞和資訊。

<!--Image references-->
[1]: ./media/security-center-using-recommendations/security-center-policy-inheritance.png
[2]: ./media/security-center-using-recommendations/scenario-roles.png
[3]: ./media/security-center-using-recommendations/select-recommendations-tile.png
[4]: ./media/security-center-using-recommendations/install-endpoint-protection.png
[5]:./media/security-center-using-recommendations/microsoft-antimalware.png
[6]: ./media/security-center-using-recommendations/provide-security-contact-details.png
[7]: ./media/security-center-using-recommendations/dismiss-recommendation.png
