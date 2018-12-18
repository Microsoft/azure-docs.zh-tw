---
title: 在 Azure 資訊安全中心監視身分識別和存取 | Microsoft Docs
description: 了解如何在 Azure 資訊安全中心使用身分識別和存取功能，來監視使用者的存取活動和身分識別相關問題。
services: security-center
documentationcenter: na
author: rkarlin
manager: mbaldwin
editor: ''
ms.assetid: 9f04e730-4cfa-4078-8eec-905a443133da
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/08/2018
ms.author: rkarlin
ms.openlocfilehash: 4a934bd69e63605fd624d06533c4e411bc94b531
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39630913"
---
# <a name="monitor-identity-and-access-in-azure-security-center-preview"></a>在 Azure 資訊安全中心監視身分識別和存取 (預覽)
這篇文章可協助您使用 Azure 資訊安全中心來監視使用者的身分識別和存取活動。

> [!NOTE]
> 監視身分識別和存取是預覽功能，只會在資訊安全中心的標準層提供使用。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。
>
>

身分識別應該是您的企業的控制台，保護您的身分識別應該是您的第一要務。 安全性周邊已從網路周邊發展到身分識別周邊。 安全性變得較不著重於防禦您的網路，而是較著重於防禦您的資料，以及管理您應用程式和使用者的安全性。 現今有越來越多資料和應用程式移至雲端，身分識別成為新的周邊。

監視您的身分識別活動，您就能夠在事件發生前採取主動式動作，或採取回應式動作以停止攻擊。 「身分識別和存取」儀表板提供您如下建議：

- 為您訂用帳戶中具有特殊權限的帳戶啟用 MFA
- 移除您訂用帳戶中具有寫入權限的外部帳戶
- 從您的訂用帳戶中移除具有特殊權限的外部帳戶

> [!NOTE]
> 如果訂用帳戶內含超過 600 個帳戶，資訊安全中心就無法針對訂用帳戶執行身分識別建議。 未執行的建議會列在「無法使用的評估」底下，後面會有相關討論。
資訊安全中心無法針對雲端解決方案提供者 (CSP) 合作夥伴的管理專員執行身分識別建議。
>
>

請參閱[建議](security-center-identity-access.md#recommendations)，以取得資訊安全中心所提供的身分識別和存取建議清單。

## <a name="monitoring-security-health"></a>監視安全性健全狀況
您可在 [資訊安全中心 – 概觀] 儀表板上監視資源的安全性狀態。 [資源] 區段是健康情況的指示器，會顯示每個資源類型的嚴重性。

您可以選取 [建議] 來檢視所有問題的清單。 在 [資源] 底下，您可以檢視「計算與應用程式」、「資料安全性」、「網路」或「身分識別和存取」的特定問題清單。 如需有關如何套用建議的詳細資訊，請參閱[實作 Azure 資訊安全中心的安全性建議](security-center-recommendations.md)。

如需身分識別和存取建議的完整清單，請參閱[建議](security-center-identity-access.md#recommendations)。

若要繼續，請選取 [資源] 或 [資訊安全中心] 主功能表下方的 [身分識別和存取]。

![資訊安全中心儀表板][1]

## <a name="monitor-identity-and-access"></a>監視身分識別和存取
[身分識別和存取] 底下有兩個索引標籤：

- **概觀**：資訊安全中心所識別的建議。
- **訂用帳戶**：訂用帳戶的清單和各自的目前安全性狀態。

![身分識別和存取][2]

### <a name="overview-section"></a>概觀區段
[概觀] 底下有建議清單。 第一個資料行會列出建議。 第二個資料行則顯示受該項建議影響的訂用帳戶總數。 第三個資料行顯示問題的嚴重性。

1. 選取建議。 建議的視窗隨即開啟，並且會顯示：

  - 建議的描述
  - 狀況不良和狀況良好的訂用帳戶清單
  - 資源清單，內含由於評估失敗而未掃描的資源，或是位於免費層中所執行訂用帳戶底下且未進行評估的資源

  ![建議的視窗][3]

1. 如需其他詳細資料，請選取清單中的訂用帳戶。

### <a name="subscriptions-section"></a>訂用帳戶區段
[訂用帳戶] 底下有訂用帳戶的清單。 第一個資料行會列出訂用帳戶。 第二個資料行會顯示每個訂用帳戶的建議總數。 第三個資料行會顯示問題的嚴重性。

![訂用帳戶的索引標籤][4]

1.  選取一個訂用帳戶。 摘要檢視隨即開啟，並有三個索引標籤：

  - **建議**：根據資訊安全中心所執行的失敗評估。
  - **已通過的評估**：資訊安全中心所執行、已通過的評估清單。
  - **無法使用的評估**：由於發生錯誤或訂用帳戶內含超過 600 個帳戶而無法執行的評估清單。

  [建議] 底下有所選訂用帳戶的建議清單和每個建議的嚴重性。

  ![所選訂用帳戶的建議][5]

1. 選取建議，以獲得建議描述、狀況不良和狀況良好的訂用帳戶清單和未掃描的資源清單。

  ![建議的描述][6]

  [已通過的評估] 底下有已通過的評估清單。  這些評估的嚴重性永遠是綠色的。

  ![已通過的評估][7]

1. 從清單中選取已通過的評估，以取得評估的描述和狀況良好訂用帳戶的清單。 狀況不良的訂用帳戶會有一個索引標籤，其中會列出所有失敗的訂用帳戶。

  ![已通過的評估][8]

## <a name="recommendations"></a>建議
請使用下表作為參考，以協助您了解可用的「身分識別和存取」建議，以及如果套用建議，每一個建議所產生的作用。

| 建議 | 說明 |
| --- | --- |
| 為訂用帳戶指定多位擁有者 | 建議您指定多位訂用帳戶擁有者，以便擁有系統管理員存取備援。 |
| 為訂用帳戶指定最多 3 位擁有者 | 建議您指定少於 3 位的訂用帳戶擁有者，以降低已遭入侵擁有者造成缺口的可能性。 |
| 為您訂用帳戶上具有擁有者權限的帳戶啟用 MFA | 建議您為所有具備系統管理員權限的訂用帳戶啟用 Multi-Factor Authentication (MFA)，以避免出現帳戶或資源缺口。 |
| 為您訂用帳戶中具有寫入權限的帳戶啟用 MFA | 建議您為所有具備寫入權限的訂用帳戶啟用 Multi-Factor Authentication (MFA)，以避免出現帳戶或資源缺口。 |
| 為您訂用帳戶中具有讀取權限的帳戶啟用 MFA | 建議您為所有具備讀取權限的訂用帳戶啟用 Multi-Factor Authentication (MFA)，以避免出現帳戶或資源缺口。 |
| 移除您訂用帳戶中具有讀取權限的外部帳戶 | 建議您移除訂用帳戶中具有讀取權限的外部帳戶，以避免出現未受監視的存取。 |
| 移除您訂用帳戶中具有寫入權限的外部帳戶 | 建議您移除訂用帳戶中具有寫入權限的外部帳戶，以避免出現未受監視的存取。 |
| 移除您訂用帳戶中具有擁有者權限的外部帳戶 | 建議您移除訂用帳戶中具有擁有者權限的外部帳戶，以避免出現未受監視的存取。 |
| 移除訂用帳戶中已淘汰的帳戶 | 建議您移除訂用帳戶中已淘汰的帳戶。 |
| 從訂用帳戶中移除具有擁有者權限的已淘汰帳戶 | 建議您移除訂用帳戶中具有擁有者權限的已淘汰帳戶。 |

## <a name="next-steps"></a>後續步驟
若要深入了解適用於其他 Azure 資源類型的建議，請參閱下列文章︰

- [保護 Azure 資訊安全中心內的機器和應用程式](security-center-virtual-machine-recommendations.md)
- [保護 Azure 資訊安全中心內的網路](security-center-network-recommendations.md)
- [保護 Azure 資訊安全中心內的 Azure SQL 服務和資料](security-center-sql-service-recommendations.md)

如要深入了解資訊安全中心，請參閱下列主題：
* [在 Azure 資訊安全中心管理和回應安全性警示](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)。 了解如何在資訊安全中心管理警示，以及回應安全性事件。
* [了解 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)。 了解不同類型的安全性警示。
* [Azure 資訊安全中心常見問題集](security-center-faq.md)。 尋找有關使用資訊安全中心常見問題的答案。


<!--Image references-->
[1]: ./media/security-center-identity-access/overview.png
[2]: ./media/security-center-identity-access/identity-dashboard.png
[3]: ./media/security-center-identity-access/select-subscription.png
[4]: ./media/security-center-identity-access/subscriptions.png
[5]: ./media/security-center-identity-access/recommendations.png
[6]: ./media/security-center-identity-access/designate.png
[7]: ./media/security-center-identity-access/passed-assessments.png
[8]: ./media/security-center-identity-access/remove.png
