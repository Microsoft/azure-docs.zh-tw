---
title: 使用 Azure 資訊安全中心管理端點保護 | Microsoft Docs
description: 了解如何管理 Azure 資訊安全中心中的端點保護問題。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 1599ad5f-d810-421d-aafc-892e831b403f
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/23/2017
ms.author: terrylan
ms.openlocfilehash: abbcb0a8e0206d78ca94520dfa81ab92506c47af
ms.sourcegitcommit: 4d90200f49cc60d63015bada2f3fc4445b34d4cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2017
ms.locfileid: "23497346"
---
# <a name="manage-endpoint-protection-issues-with-azure-security-center"></a>使用 Azure 資訊安全中心管理端點保護
Azure 資訊安全中心會監視反惡意程式碼軟體的狀態，並在 [端點保護問題] 刀鋒視窗下報告此狀態。 資訊安全中心會強調問題所在，例如偵測到威脅和防護不足，這些問題會造成虛擬機器 (VM) 和電腦容易遭受反惡意程式碼軟體威脅。 您可以使用 [端點保護問題] 的資訊，找出可解決找出的問題的方案。

資訊安全中心會報告下列端點保護問題：

- Azure VM 上未安裝端點保護 – Azure VM 未安裝支援的反惡意程式碼軟體解決方案。
- 非 Azure 電腦上未安裝端點保護 – 非 Azure 電腦未安裝支援的反惡意程式碼軟體。
- 端點保護健康情況：

   - 簽章已過期 - VM 和電腦上已安裝反惡意程式碼軟體解決方案，但是該解決方案沒有最新的反惡意程式碼軟體簽章。
   - 沒有即時保護 - VM 和電腦上已安裝反惡意程式碼軟體解決方案，但是並未設定即時保護。   該服務可能已停用，或者資訊安全中心可能因為不支援該解決方案而無法取得狀態。 如需支援的解決方案清單，請參閱[夥伴整合](security-center-partner-integration.md)。
   - 未回報 – 已安裝反惡意程式碼軟體解決方案，但是未報告資料。
   - 未知 - 已安裝反惡意程式碼軟體解決方案，但是其狀態為未知或報告未知錯誤。

   > [!NOTE]
   > 如需與資訊安全中心之端點保護安全性解決方案的清單，請參閱[整合安全性解決方案](security-center-partner-integration.md#integrated-azure-security-solutions)。
   >
   >

## <a name="implement-the-recommendation"></a>實作建議
端點保護會以建議的形式在資訊安全中心中呈現。  如果您的環境容易遭受反惡意程式碼軟體威脅，在 [建議] 下和 [計算] 下會顯示這項建議。 若要查看**端點保護問題儀表板**，您必須遵循計算工作流程。

在此範例中，我們使用 [計算]。  我們將探討如何在 Azure VM 和非 Azure 電腦上安裝反惡意程式碼軟體。

## <a name="install-antimalware-on-azure-vms"></a>在 Azure VM 上安裝反惡意程式碼軟體

1. 在 [資訊安全中心] 主功能表或 [概觀] 下選取 [計算]。

   ![選取 [計算]][1]

2. 在 [計算] 下，選取 [端點保護問題]。 [端點保護問題] 儀表板隨即開啟。

   ![選取 [端點保護問題]][2]

   儀表板的頂端提供：

   - 已安裝的端點保護提供者 - 列出資訊安全中心識別的不同提供者。
   - 已安裝的端點保護健康情況狀態 - 顯示已安裝端點保護解決方案的電腦和 VM 健康情況狀態。 該圖表顯示健康的 VM 和電腦數量，以及保護不足的數量。
   - 偵測到惡意程式碼 – 顯示資訊安全中心報告偵測到惡意程式碼的 VM 和電腦數量。
   - 遭攻擊的電腦 – 顯示資訊安全中心報告遭到惡意程式碼攻擊的 VM 和電腦數量。

   儀表板底部有端點保護問題的清單，包含下列資訊：  

   - **總計** - 受此問題影響的 VM 和電腦數量。
   - 受此問題影響的 VM 和電腦累計數量的橫條。 橫條的色彩表示優先順序：

      - 紅色 - 高優先順序，應立即處理
      - 橘色 - 中等優先順序，應儘速處理

3. 選取 [Azure VM 上未安裝端點保護]。

   ![選取 [Azure VM 上未安裝端點保護]][3]

4. 在 [Azure VM 上未安裝端點保護] 下，有未安裝反惡意程式碼軟體的 Azure VM 清單。  您可以在清單中選擇在所有 VM 上安裝反惡意程式碼軟體，也可以按一下特定的 VM，選取個別 VM 安裝反惡意程式碼軟體。
5. 在 [選取端點保護] 下，選取想要使用的端點保護解決方案。 在此範例中，選取 [Microsoft Antimalware] 。
6. 將會顯示 Endpoint Protection 解決方案的其他相關資訊。 選取 [建立] 。

## <a name="install-antimalware-on-non-azure-computers"></a>在非 Azure 電腦上安裝反惡意程式碼軟體

1. 返回 [端點保護問題]，並選取 [非 Azure 電腦上未安裝端點保護]。

   ![選取 [非 Azure 電腦上未安裝端點保護]][4]

2. 在 [非 Azure 電腦上未安裝端點保護] 下，選取工作區。 針對該工作區篩選的 Log Analytics 搜尋查詢會顯示，並列出遺漏反惡意程式碼軟體的電腦。 從清單中選取電腦以了解更多資訊。

   ![Log Analytics 搜尋][5]

會出現僅針對該電腦篩選出的其他搜尋結果。

  ![Log Analytics 搜尋][6]

> [!NOTE]
> 建議為所有 VM 和電腦佈建端點保護，以便識別和移除病毒、間諜軟體及其他惡意軟體。
>
>

## <a name="next-steps"></a>後續步驟
本文說明了如何實作資訊安全中心建議的「安裝端點保護」。 若要深入了解如何在 Azure 中啟用反惡意程式碼軟體，請參閱下列文件：

* [適用於雲端服務和虛擬機器的 Microsoft Antimalware](../security/azure-security-antimalware.md) -- 了解如何部署 Microsoft Antimalware。

若要深入了解資訊安全中心，請參閱下列文件：

* [設定 Azure 資訊安全中心的安全性原則](security-center-policies.md) -- 了解如何設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 尋找有關 Azure 安全性與相容性的部落格文章。

<!--Image references-->
[1]:./media/security-center-install-endpoint-protection/compute.png
[2]:./media/security-center-install-endpoint-protection/endpoint-protection-issues.png
[3]:./media/security-center-install-endpoint-protection/install-endpoint-protection.png
[4]:./media/security-center-install-endpoint-protection/endpoint-protection-issues-computers.png
[5]:./media/security-center-install-endpoint-protection/log-search.png
[6]:./media/security-center-install-endpoint-protection/info-filtered-to-computer.png
