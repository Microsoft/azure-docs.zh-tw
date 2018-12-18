---
title: 在 Azure 資訊安全中心修復安全性設定 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議的「修復安全性設定」。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 991d41f5-1d17-468d-a66d-83ec1308ab79
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/10/2018
ms.author: terrylan
ms.openlocfilehash: 3c56abcec37bb6abcb77ec8cc443b0656bd69932
ms.sourcegitcommit: df50934d52b0b227d7d796e2522f1fd7c6393478
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38990790"
---
# <a name="remediate-security-configurations-in-azure-security-center"></a>在 Azure 資訊安全中心修復安全性設定
Azure 資訊安全中心會針對可能讓虛擬機器 (VM) 和電腦更容易遭受攻擊的設定，每天分析虛擬機器和電腦的作業系統。 當您的 OS 設定不符合建議的安全性設定規則時，資訊安全中心會建議您解決這些弱點，並建議變更設定以解決這些弱點。

如需受監視之特定設定的詳細資訊，請參閱[建議的設定規則清單](https://gallery.technet.microsoft.com/Azure-Security-Center-a789e335) \(英文\)。 若要深入了解如何自訂安全性設定評估，請參閱[在 Azure 資訊安全中心 (預覽) 自訂 OS 安全性設定](security-center-customize-os-security-config.md)。

## <a name="implement-the-recommendation"></a>實作建議
「修復安全性設定」會以建議的形式在資訊安全中心中呈現。 建議會顯示在 [建議] > [計算及應用程式] 下方。

此範例涵蓋 [計算及應用程式] 下方的「修復安全性設定」建議。
1. 在 [資訊安全中心] 的左窗格中，選取 [計算及應用程式]。  
  [計算及應用程式] 視窗隨即開啟。

   ![修復安全性設定][1]

2. 選取 [修復安全性設定]。  
  [安全性設定] 視窗隨即開啟。

   ![[安全性設定] 視窗隨即開啟。][2]

  儀表板上半部會顯示：

  - **失敗的規則 (依嚴重性)**：規則總數，依 OS 設定在 VM 和電腦上發生故障的嚴重性。
  - **失敗的規則 (依類型)**：規則總數，依 OS 設定在 VM 和電腦上發生故障的類型。
  - **失敗的 Windows 規則**：您的 Windows OS 設定失敗的規則總數。
  - **失敗的 Linux 規則**：您的 Linux OS 設定失敗的規則總數。

  儀表板下半部會列出 VM 和電腦的所有失敗規則，以及缺少更新的嚴重性。 此清單包含下列元素：

  - **CCEID**：規則的 CCE 唯一識別碼。 資訊安全中心會使用一般設定列舉 (CCE) 將唯一識別碼指派給設定規則。
  - **名稱**：失敗規則的名稱。
  - **規則類型**：[登錄機碼]、[安全性原則] [稽核原則] 或 [IIS] 規則類型。
  - **VM 及電腦數目**：套用失敗規則的 VM 和電腦總數。
  - **規則嚴重性**：[嚴重]、[重要] 或 [警告] 的 CCE 值。
  - **狀態**：建議的目前狀態：

    - **未處理**：建議尚未處理。
    - **進行中**：正在將建議套用到資源，且您不需要採取任何動作。
    - **已解決**：已套用建議。 若問題已解決，該項目會呈現暗灰色。

3. 若要檢視失敗規則的詳細資訊，請在清單中選取它。

   ![失敗的設定規則的詳細檢視][3]

   詳細檢視會顯示下列資訊：

   - **名稱**：規則的名稱。
   - **CCEID**：規則的 CCE 唯一識別碼。
   - **OS 版本** - VM 或電腦的 OS 版本。
   - **規則嚴重性**：[嚴重]、[重要] 或 [警告] 的 CCE 值。
   - **完整描述** : 規則的描述。
   - **弱點**：不套用規則的弱點或風險說明。
   - **潛在影響**：套用規則時的業務影響。
   - **對策**：修復步驟。
   - **預期值**：資訊安全中心對照規則分析 VM OS 設定時的預期值。
   - **實際值**：對照規則分析 VM OS 設定後的傳回值。
   - **規則作業**：資訊安全中心對照規則分析 VM OS 設定時使用的規則作業。

4. 在詳細檢視視窗頂端，選取 [搜尋]。  
  搜尋會開啟一個工作區清單，其中的 VM 和電腦的所選安全性設定不符。 只有在已選取適用於連線至不同工作區之多個 VM 的規則時，才會顯示 [工作區選擇]。

   ![列出的工作區][4]

5. 選取工作區。  
  Log Analytics 搜尋查詢會開啟並只篩出安全性設定不符的工作區。

   ![具有作業系統漏洞的工作區][5]

6. 選取清單中的電腦。  
  會出現僅針對該電腦篩選出的新搜尋結果。

   ![所選電腦的詳細資訊][6]

## <a name="next-steps"></a>後續步驟
本文說明如何實作資訊安全中心建議的「修復安全性設定」。 若要深入了解如何自訂安全性設定評估，請參閱[在 Azure 資訊安全中心 (預覽) 自訂 OS 安全性設定](security-center-customize-os-security-config.md)。

若要檢閱受監視的特定設定，請參閱[建議的設定規則清單](https://gallery.technet.microsoft.com/Azure-Security-Center-a789e335) \(英文\)。 資訊安全中心會使用一般設定列舉 (CCE) 將唯一識別碼指派給設定規則。 如需詳細資訊，請移至 [CCE](https://nvd.nist.gov/cce/index.cfm) 網站。

若要深入了解資訊安全中心，請參閱下列資源：

* 如需支援的 Windows 與 Linux VM 清單，請參閱 [Azure 資訊安全中心支援的平台](security-center-os-coverage.md)。
* 如果要了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則，請參閱[在 Azure 資訊安全中心設定安全性原則](security-center-policies.md)。
* 如果要了解建議如何協助保護您的 Azure 資源，請參閱[管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md)。
* 如果要了解如何監視 Azure 資源的健康情況，請參閱 [Azure 資訊安全中心的安全性健康情況監視](security-center-monitoring.md)。
* 如果要了解如何管理及回應安全性警示，請參閱[管理及回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md)。
* 如果要了解如何監視合作夥伴解決方案的健康情況，請參閱[使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md)。
* 如需關於使用服務的常見問題解答，請參閱 [Azure 資訊安全中心常見問題集](security-center-faq.md)。
* 如需有關 Azure 安全性與合規性的部落格文章，請參閱 [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) \(英文\)。

<!--Image references-->
[1]: ./media/security-center-remediate-os-vulnerabilities/compute-blade.png
[2]:./media/security-center-remediate-os-vulnerabilities/os-vulnerabilities.png
[3]: ./media/security-center-remediate-os-vulnerabilities/vulnerability-details.png
[4]: ./media/security-center-remediate-os-vulnerabilities/search.png
[5]: ./media/security-center-remediate-os-vulnerabilities/log-search.png
[6]: ./media/security-center-remediate-os-vulnerabilities/search-results.png
