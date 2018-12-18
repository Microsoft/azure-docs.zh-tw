---
title: 什麼是 Azure 資訊安全中心？| Microsoft Docs
description: 了解 Azure 資訊安全中心、其主要功能及運作方式。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: 45b9756b-6449-49ec-950b-5ed1e7c56daa
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/10/2018
ms.author: rkarlin
ms.openlocfilehash: 1b0a6c6d6daa686404021afb2e10b25bb9f54191
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319379"
---
# <a name="what-is-azure-security-center"></a>什麼是 Azure 資訊安全中心？
Azure 資訊安全中心為混合式雲端工作負載提供統一的安全性管理和進階威脅防護。 使用資訊安全中心，您可以在工作負載中套用安全性原則、限制暴露於威脅的可能性，以及偵測和回應攻擊。

為何使用資訊安全中心？

- **集中式原則管理** – 集中管理所有混合式雲端工作負載的安全性原則，確保符合公司或法規的安全性需求。
- **連續安全性評估** – 監視機器、網路、儲存體與資料服務以及應用程式的安全性狀態，以找出潛在的安全性問題。
- **的建議** – 運用已排定優先順序和可操作的安全性建議，在攻擊者入侵之前修正安全性弱點。
- **已排定優先順序的警示和事件** - 藉由已排定優先順序的安全性警示和事件，首先專注處理最嚴重的威脅。
- **進階雲端防禦** – 透過 Just-In-Time 存取管理連接埠及在 VM 上執行的自適性應用程式控制項來降低威脅。
- **整合式安全性解決方案** - 收集、搜尋和分析各種來源 (包括已連線的合作夥伴解決方案) 的安全性資料。

[資訊安全中心 - 概觀] 可讓您快速檢視您 Azure 與非 Azure 工作負載的安全性狀態、探索並評定工作負載的安全性，並識別和降低風險。 內建儀表板可供立即深入掌握需要注意的安全性警示和弱點。

![概觀][1]

## <a name="centralized-policy-management"></a>集中式管理原則
[原則與合規性] 區段 (如上所示) 提供有關訂用帳戶涵蓋範圍、原則合規性與安全性狀態的快速資訊。

### <a name="subscription-coverage"></a>訂用帳戶涵蓋範圍
此區段會顯示您具有存取權 (讀取或寫入) 的訂用帳戶總數，以及正在執行訂用帳戶的資訊安全中心涵蓋範圍層級 (「標準」或「免費」)：

- **涵蓋在內 (標準)** – 涵蓋的訂用帳戶會在資訊安全中心提供之最大保護層級下執行
- **涵蓋在內 (免費)** – 涵蓋的訂用帳戶會在資訊安全中心提供之有限的免費保護層級下執行
- **未涵蓋** – 此狀態中的訂用帳戶未受資訊安全中心監視

選取圖表即會開啟 [涵蓋範圍] 視窗。 選取索引標籤 ([未涵蓋]、[基本的涵蓋範圍]，或 [標準的涵蓋範圍]) 會提供每種狀態的訂用帳戶清單。 在其中一個索引標籤下選取訂用帳戶，可提供有關訂用帳戶的其他資訊。 此資訊可讓您找出訂用帳戶的擁有者並與他們連絡，以啟用資訊安全中心，或增加訂用帳戶的涵蓋範圍。

![資訊安全中心涵蓋範圍][9]

### <a name="policy-compliance"></a>原則相容性
原則合規性是由指派之所有原則的合規性因素所決定。 管理群組、訂用帳戶或工作區的整體合規性分數是指派的加權平均值。 單一指派中原則數目的加權平均因子，以及套用指派的資源數目。

例如，如果訂用帳戶有兩個 VM 以及已指派五個原則的方案，則您的訂用帳戶中有十個評估。 如果其中一個 VM 不符合兩個原則，您訂用帳戶指派的整體合規性分數為 80%。

此區段會顯示您的整體合規性比例和最不符合規範的訂用帳戶。 選取 [顯示環境的原則合規性] 將會開啟 [原則管理] 視窗。 [原則管理] 會顯示管理群組、訂用帳戶與工作區的階層式結構。 在這裡，您可以透過選擇訂用帳戶或管理群組來管理您的安全性原則。

![原則管理][10]

安全性原則會定義工作負載的所需設定，並且協助確保符合公司或法規安全性需求。 在資訊安全中心，您可以定義原則，並依照工作負載的類型或資料的敏感性量身打造這些原則，判斷哪個原則控制資訊安全中心監視和建議。 您可以透過按一下管理群組或訂用帳戶，在資訊安全中心中編輯安全性原則。 您也可以使用 Azure 原則建立新的定義、定義額外的原則，並在管理群組指派原則。

選取 [編輯設定 >]，在訂用帳戶、管理群組、資源群組或工作區層級編輯下列資訊安全中心設定：

- **資料收集**：決定代理程式佈建和安全性[資料收集](security-center-enable-data-collection.md)設定。
- **電子郵件通知**：決定安全性連絡人和[電子郵件通知](security-center-provide-security-contact-details.md)設定。
- **定價層**：定義免費或標準[定價選取項目](security-center-pricing.md)。 您選擇的一層將決定範圍中的資源可用的資訊安全中心功能。
- **編輯安全性設定**：可讓您檢視及修改資訊安全中心所評定的 OS 設定，以找出潛在的安全性弱點。

如需詳細資訊，請參閱[整合資訊安全中心安全性原則與 Azure 原則](security-center-azure-policy.md)。

### <a name="manage-and-govern-your-security-posture"></a>控管您的安全性狀態
[原則與合規性] 下的儀表板右側為您提供了可立即採取動作以改善您整體安全性狀態的見解。 範例包括：

- 定義及指派資訊安全中心原則，以檢閱及追蹤安全性標準的合規性
- 將資訊安全中心安全性警示提供給 SIEM 連接器
- 經過一段時間的原則合規性

## <a name="continuous-security-assessment"></a>持續安全性評估
[資訊安全中心 - 概觀] 下的 [資源安全性防護] 區段提供了資源安全性防護的快速檢視，顯示已識別的問題數目與每個資源類型的安全性狀態。 連續評估可協助您找出潛在的安全性問題，例如遺漏安全性更新或已公開網路連接埠的系統。

### <a name="secure-score"></a>安全分數
Azure 資訊安全中心的安全分數會檢閱您的安全性建議，並為您確定優先順序，以便您了解要先執行哪些建議，進而協助您找出最嚴重的安全性弱點，以便優先排定調查。 安全分數是一種測量工具，可協助您強化您的安全性，以達到安全的工作負載。 如需詳細資訊，請參閱 [Azure 資訊安全中心中的安全分數](security-center-secure-score.md)。

### <a name="health-monitoring"></a>健康狀況監視
在 [資源健康情況監視] 下選取資源類型，可提供資源與任何已識別之弱點的清單。 資源類型包括計算和應用程式、網路、資料與儲存體，以及身分識別與存取權。

我們選取了 [計算及應用程式]。 [計算] 下方有四個索引標籤：

- **概觀**：資訊安全中心所識別的監視和建議。
- **VM 和電腦**：列出您的 VM、電腦和各項目目前的安全性狀態。
- **雲端服務**︰列出資訊安全中心所監視的 Web 和背景工作角色。
- **App Service (預覽)**：列出您的 Web 應用程式與應用程式服務環境和各項目目前的安全性狀態。

![安全性健康情況監視][3]

如需詳細資訊，請參閱[安全性健康情況監視](security-center-monitoring.md)。

## <a name="actionable-recommendations"></a>可操作的建議
資訊安全中心會分析 Azure 和非 Azure 資源的安全性狀態，以識別潛在的安全性弱點。 選取 [資源] 下的 [建議] 會提供已設定優先順序的安全性建議清單引導您完成處理安全性問題的程序。

![建議][4]

如需詳細資訊，請參閱[管理安全性建議](security-center-recommendations.md)。

### <a name="most-prevalent-recommendations"></a>最普遍的建議
[資源] 下的儀表板右側為您提供了大量資源最普遍存在的建議清單。 最普遍的建議強調了您應該注意的地方。 選取向右鍵可提供最高影響建議。

![最普遍的建議][11]

這是在您環境中最具影響力的單一建議。 解決此建議將會大幅改進您的合規性。 在此範例中，建議是「套用磁碟加密」。 選取 [改進您的合規性] 會提供建議的描述，以及受影響資源的清單。

![套用磁碟加密][12]

## <a name="threat-protection"></a>威脅保護
此區域會提供在您的資源上偵測到之安全性警示的可見度，以及這些警示的嚴重性層級。

### <a name="prioritized-alerts-and-incidents"></a>已排定優先順序的警示和事件
資訊安全中心會使用進階分析和全球威脅情報，偵測傳入的攻擊和缺口後的活動。 系統會設定警示的優先順序並分組成為事件，協助您先將焦點放在最嚴重的威脅。 您也可以建立自己的自訂安全性警示。

選取 [安全性警示 (依嚴重性)] 或 [一段時間的安全性警示] 會提供有關警示的詳細資訊。

![已排定優先順序的警示和事件][7]

您可以透過視覺化、互動式調查體驗，快速評定攻擊的範圍和影響，並使用預先定義或隨選查詢更深入探索安全性資料。

如需詳細資訊，請參閱[管理及回應安全性警示](security-center-managing-and-responding-alerts.md)。

儀表板右側提供的資訊可協助您優先排定要處理的警示，並了解常見弱點的位置：

- **受到最多攻擊的資源**：具有最多警示數目的特定資源
- **最常見的警示**：影響資源數目最的警示類型

## <a name="just-in-time-vm-access"></a>Just-In-Time 虛擬機器存取
利用 Just-In-Time 控制存取減少網路攻擊面，以管理 Azure 虛擬機器上的連接埠，大幅減少暴力密碼破解攻擊和其他網路攻擊。

![Just-In-Time 虛擬機器存取][5]

指定使用者如何連線到虛擬機器的規則。 如有需要，可以從資訊安全中心或透過 PowerShell 要求存取權。 只要要求符合規則，系統會針對要求的時間自動授與存取權。

如需詳細資訊，請參閱[使用 Just-In-Time 管理虛擬機器存取](security-center-just-in-time.md)。

## <a name="adaptive-application-controls"></a>自適性應用程式控制
藉由套用依特定 Azure 工作負載調整並由機器學習服務提供的允許清單建議，來封鎖惡意程式碼和其他不想要的應用程式。

![自適性應用程式控制][6]

檢閱並按一下以套用資訊安全中心所產生的建議應用程式允許清單設定規則，或編輯已經設定的規則。

如需詳細資訊，請參閱[自適性應用程式控制](security-center-adaptive-application.md)。

## <a name="integrate-your-security-solutions"></a>整合安全性解決方案
在資訊安全中心，您可以收集、搜尋和分析各種來源 (包括已連線的合作夥伴解決方案，例如網路防火牆和其他 Microsoft 服務) 的安全性資料。

![整合安全性解決方案][8]

如需詳細資訊，請參閱[整合安全性解決方案](security-center-partner-integration.md)。

## <a name="next-steps"></a>後續步驟

- 若要開始使用資訊安全中心，您需要 Microsoft Azure 訂用帳戶。 如果您沒有訂用帳戶，可以註冊[免費試用](https://azure.microsoft.com/free/)。
- 資訊安全中心的定價層已經由 Azure 訂用帳戶啟用。 若要利用進階安全性管理和威脅偵測功能，您必須升級為標準定價層。 標準層的前 60 天免費。 如需詳細資訊，請參閱[資訊安全中心定價頁面](https://azure.microsoft.com/pricing/details/security-center/)。
- 如果您已準備好立即啟用資訊安全中心的標準層，[快速入門：將 Azure 訂用帳戶上架到資訊安全中心的標準層](security-center-get-started.md)會逐步引導您完成步驟。


<!--Image references-->
[1]: ./media/security-center-intro/overview.png
[3]: ./media/security-center-intro/compute.png
[4]: ./media/security-center-intro/recommendations.png
[5]: ./media/security-center-intro/just-in-time-vm-access.png
[6]: ./media/security-center-intro/adaptive-app-controls.png
[7]: ./media/security-center-intro/security-alerts.png
[8]: ./media/security-center-intro/security-solutions.png
[9]: ./media/security-center-intro/coverage.png
[10]: ./media/security-center-intro/policy-management.png
[11]: ./media/security-center-intro/highest-impact.png
[12]: ./media/security-center-intro/apply-disk-encryption.png
