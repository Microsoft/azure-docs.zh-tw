---
title: 在 Azure 資訊安全中心管理安全性警示 | Microsoft Docs
description: 本文件可協助您使用「Azure 資訊安全中心」功能來管理及回應安全性警示。
services: security-center
documentationcenter: na
author: terrylan
manager: mbaldwin
editor: ''
ms.assetid: b88a8df7-6979-479b-8039-04da1b8737a7
ms.service: security-center
ms.topic: hero-article
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/30/2017
ms.author: yurid
ms.openlocfilehash: d088223aa2ea40d5bb9cf0e492e87ef054d86348
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34365354"
---
# <a name="managing-and-responding-to-security-alerts-in-azure-security-center"></a>管理及回應 Azure 資訊安全中心的安全性警示
本文件可協助您使用 Azure 資訊安全中心來管理及回應安全性警示。

> [!NOTE]
> 若要啟用進階偵測，請升級至 Azure 資訊安全中心標準。 提供 60 天的免費試用。 若要升級，請選取[安全性原則](security-center-policies.md)中的 [定價層]。 若要深入了解，請參閱 [Azure 資訊安全中心價格](security-center-pricing.md)。
>
>

## <a name="what-are-security-alerts"></a>什麼是安全性警示：
資訊安全中心會自動收集、分析及整合您 Azure 資源、網路和已連線的合作夥伴解決方案 (例如防火牆和端點保護解決方案) 的記錄檔資料，來偵測真正的威脅並減少誤判情形。 「資訊安全中心」會顯示優先安全性警示清單，以及需要您快速調查問題的資訊，和如何修復攻擊行為的建議。


> [!NOTE]
> 如需資訊安全中心偵測功能運作方式的詳細資訊，請閱讀 [Azure 資訊安全中心的偵測功能](security-center-detection-capabilities.md)。
>
>

## <a name="managing-security-alerts"></a>管理安全性警示
您可以查看 [安全性警示]  圖格來檢視目前的警示。 遵循下列步驟來查看有關每個警示的更多詳細資訊：

1. 您會在 [資訊安全中心] 儀表板看到 [安全性警示] 圖格。

    ![資訊安全中心的 [安全性警示] 圖格](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig1-ga.png)

2. 按一下圖格，即可開啟 [安全性警示] 來查看有關警示的更多詳細資料。

   ![資訊安全中心內的安全性警示](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig2-ga.png)

此頁面底部有每個警示的詳細資料。 如要為警示排序，請按一下您要做為排序依據的的資料行。 下列為每個資料行的定義：

* **描述**：警示的簡短說明。
* **計數**：在特定一天偵測到這個特定類型的所有警示清單。
* **偵測者**：負責觸發警示的服務。
* **日期**：事件發生的日期。
* **狀態**：該警示目前的狀態。 狀態分為兩種：
  * **使用中**：已偵測到安全性警示。
* **嚴重性**：嚴重性層級，分為高、中或低。

> [!NOTE]
> 資訊安全中心產生的安全性警示也會出現在 Azure 活動記錄之下。 如需有關如何存取 Azure 活動記錄的詳細資訊，請參閱[檢視活動記錄以稽核對資源的動作](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit)。
>

### <a name="filtering-alerts"></a>篩選警示
您可以根據日期、狀態及嚴重性來篩選警示。 如果您需要縮小顯示的安全性警示檢視範圍，篩選警示會相當有用。 例如，您可能想確認在過去 24 小時發生的安全性警示，因為您正在調查系統中可能的入侵行動。

1. 按一下 [安全性警示] 上的 [篩選]。 即會開啟 [篩選] 供您選取想要查看的日期、狀態和嚴重性值。

    ![篩選資訊安全中心的警示](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig3-2017.png)

### <a name="respond-to-security-alerts"></a>回應安全性警示
選取一個安全性警示以深入了解觸發警示的事件；如果發現項目，您需要進行一些步驟來阻止攻擊。 安全性警示會依類型及日期區分。 按一下安全性警示會開啟頁面，其中包括已分組的警示清單。

![對於 Azure 資訊安全中心的安全性警示的回應](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig5-ga.png)

在此案例中，所觸發的警示是關於可疑的遠端桌面通訊協定 (RDP) 活動。 第一個資料行顯示哪些資源遭到攻擊；第二個資料行顯示資源遭受攻擊的次數；第三個資料行顯示攻擊的時間；第四個資料行顯示警示的狀態；而第五個資料行顯示攻擊的嚴重性。 在檢閱這項資訊後，按一下遭到攻擊的資源。

![對於如何處理 Azure 資訊安全中心的安全性警示的建議](./media/security-center-managing-and-responding-alerts/security-center-managing-and-responding-alerts-fig6-ga.png)

在 [說明] 欄位中，您會找到關於這個事件的更多詳細資料。 這些額外的詳細資料可供深入了解什麼會觸發安全性警示、目標資源、來源 IP 位址 (若適用)，以及有關如何補救的建議。  在某些情況下，來源 IP 位址會是空的 (不適用)，因為並非所有的 Windows 安全性事件記錄檔都包含 IP 位址。

資訊安全中心會根據安全性警示，建議您不同的補救方法。 在某些情況下，您可能必須使用其他的 Azure 功能來實作建議的補救方法。 例如，這個攻擊的補救方法是使用[網路 ACL](../virtual-network/virtual-networks-acl.md) 或[網路安全性群組](../virtual-network/security-overview.md#security-rules)規則，將產生此攻擊的 IP 位址列入封鎖清單。 如需不同警示類型的詳細資訊，請閱讀 [Azure 資訊安全中心不同類型的安全性警示](security-center-alerts-type.md)。

> [!NOTE]
> 資訊安全中心已向有限預覽發行一組新的偵測，可運用通用稽核架構的 auditd 記錄來偵測 Linux 電腦上的惡意行為。 請將含有您的訂用帳戶識別碼的電子郵件傳送給[我們](mailto:ASC_linuxdetections@microsoft.com)，以加入預覽。


## <a name="see-also"></a>另請參閱
在本文件中，您了解到如何在資訊安全中心設定安全性原則。 如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心處理安全性事件](security-center-incident.md)
* [Azure 資訊安全中心的偵測功能](security-center-detection-capabilities.md)
* [Azure 資訊安全中心規劃和操作指南](security-center-planning-and-operations-guide.md)
* [Azure 資訊安全中心常見問題集](security-center-faq.md) – 尋找使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 尋找有關 Azure 安全性與相容性的部落格文章。
