---
title: Azure 資訊安全中心教學課程 - 定義和評估安全性原則 | Microsoft Docs
description: Azure 資訊安全中心教學課程 - 定義和評估安全性原則
services: security-center
documentationcenter: na
author: rkarlin
manager: mbaldwin
editor: ''
ms.assetid: 2d248817-ae97-4c10-8f5d-5c207a8019ea
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/30/2018
ms.author: rkarlin
ms.openlocfilehash: fcd3c2a95cea0a838fc16149a0a74fad95ea3300
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44027056"
---
# <a name="tutorial-define-and-assess-security-policies"></a>教學課程：定義和評估安全性原則
資訊安全中心可讓您使用安全性原則來定義工作負載所需的設定，協助確保符合公司或法規安全性需求。 您為 Azure 訂用帳戶定義原則，並針對工作負載類型或資料敏感性進行調整後，資訊安全中心即可對您的計算、應用程式、網路、資料和儲存體以及身分識別和存取資源，提供安全性建議。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 設定安全性原則
> * 評估資源的安全性

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="prerequisites"></a>必要條件
若要逐步執行本教學課程中涵蓋的功能，您必須是在資訊安全中心的標準定價層。 前 60 天可以免費試用資訊安全中心標準定價層。 [將 Azure 訂用帳戶上架到資訊安全中心標準定價層](security-center-get-started.md)快速入門會為您逐步解說如何升級至「標準」定價層。

## <a name="configure-security-policy"></a>設定安全性原則
資訊安全中心會為每個 Azure 訂用帳戶自動建立預設安全性原則。 安全性原則是由一些建議所組成，您可以根據該訂用帳戶的安全性需求來開啟或關閉原則。 若要變更預設安全性原則，您必須是該訂用帳戶的擁有者、參與者或安全性管理員。

1. 在 [資訊安全中心] 主功能表上，選取 [安全性原則]。
2. 選取您要使用的訂用帳戶。

  ![安全性原則](./media/tutorial-security-policy/tutorial-security-policy-fig1.png)  

3. 在 [計算和應用程式]、[網路] 和 [資料] 底下，將您想要監視的每個安全性組態設定為 [開啟]。 資訊安全中心會持續評估環境的設定，當有弱點存在時，資訊安全中心就會產生安全性建議。 如果不建議使用此安全性設定或此安全性設定不相關，請選取 [關閉]。 例如，在開發/測試環境中，您可能不需要與生產環境相同層級的安全性。 選取您的環境適用的原則之後，按一下 [儲存]。

  ![安全性組態](./media/tutorial-security-policy/tutorial-security-policy-fig6.png)  

等待資訊安全中心處理好這些原則，然後產生建議。 雖然有些設定 (例如系統更新和 OS 設定) 可能需要多達 12 個小時的時間，但網路安全性群組和加密設定卻可幾乎立即進行評估。 一旦在資訊安全中心儀表板中看到建議，您就可以繼續進行下一個步驟。

## <a name="assess-security-of-resources"></a>評估資源的安全性
1. 資訊安全中心會根據已啟用的安全性原則，視需要提供一組安全性建議。 您應該從檢閱虛擬機器和電腦建議開始。 在 [資訊安全中心] 儀表板上，選取 [概觀]，然後選取 [計算及應用程式]。

  ![計算](./media/tutorial-security-policy/tutorial-security-policy-fig2.png)

  以紅色 (高優先順序) 設定建議的優先順序，藉此檢閱每個建議。 其中有些建議可直接從資訊安全中心實作補救措施，例如[端點保護問題](https://docs.microsoft.com/azure/security-center/security-center-install-endpoint-protection)。 而其他建議只有套用補救措施的方針，例如遺漏磁碟加密建議。

2. 在您處理所有計算相關建議之後，您應該繼續處理下一個工作負載：網路。 在 [資訊安全中心] 儀表板上，按一下 [概觀]，然後按一下 [網路]。

  ![網路功能](./media/tutorial-security-policy/tutorial-security-policy-fig3.png)

  [網路建議] 頁面中有網路設定、網際網路面向端點和網路拓撲的安全性問題清單。 就如同 [計算及應用程式]，有些網路建議會提供整合式補救措施，而其他網路建議則不會提供補救措施。

3. 在您處理所有網路相關建議之後，您應該繼續處理下一個工作負載：儲存體和資料。 在 [資訊安全中心] 儀表板上，按一下 [概觀]，然後按一下 [資料和儲存體]。

  ![資料資源](./media/tutorial-security-policy/tutorial-security-policy-fig4.png)

  [資料資源] 頁面包含以下列項目為主的建議：為 Azure SQL 伺服器和資料庫啟用稽核、為 SQL 資料庫啟用加密，以及啟用 Azure 儲存體帳戶加密。 如果您沒有這些工作負載，就看不到任何建議。 就如同 [計算及應用程式]，有些 SQL 和儲存體建議會提供整合式補救措施，而其他 SQL 和儲存體建議則不會提供補救措施。

4. 在您處理所有資料和儲存體相關建議之後，您應該繼續處理下一個工作負載：身分識別和存取。 在 [資訊安全中心] 儀表板上，按一下 [概觀]，然後按一下 [身分識別和存取]。

  ![身分識別和存取](./media/tutorial-security-policy/tutorial-security-policy-fig5.png)

  [身分識別和存取] 頁面包含一些建議，例如：

   - 為您訂用帳戶中具有特殊權限的帳戶啟用 MFA
   - 移除您訂用帳戶中具有寫入權限的外部帳戶
   - 從您的訂用帳戶中移除具有特殊權限的外部帳戶

## <a name="clean-up-resources"></a>清除資源
此集合中的其他快速入門和教學課程會以本快速入門為基礎。 如果您打算繼續處理後續的快速入門和教學課程，請繼續執行標準層，並保持將自動佈建維持為啟用狀態。 如果您不打算繼續，或是要返回免費層：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 選取您需要返回免費層的訂用帳戶或原則。 [安全性原則] 隨即開啟。
3. 在 [原則元件] 下，選取 [定價層]。
4. 選取 [免費] 以將訂用帳戶從標準層變更為免費層。
5. 選取 [ **儲存**]。

如果您需要停用自動佈建：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 選取您想要停用自動佈建的訂用帳戶。
3. 在 [安全性原則 - 資料收集] 下，選取 [上架] 底下的 [關閉] 以停用自動佈建。
4. 選取 [ **儲存**]。

>[!NOTE]
> 停用自動佈建不會從已佈建代理程式的 Azure VM 移除 Microsoft Monitoring Agent。 停用自動佈建會限制對資源的安全性監視。
>

## <a name="next-steps"></a>後續步驟
在此教學課程中，您已了解如何使用資訊安全中心為您的工作負載進行基本的原則定義和安全性評估，例如：

> [!div class="checklist"]
> * 安全性原則設定可確保符合公司或法規安全性需求
> * 您的計算、網路、SQL 和儲存體及應用程式資源的安全性評估

前進到下一個教學課程，以了解如何使用資訊安全中心來保護您的資源。

> [!div class="nextstepaction"]
> [保護您的資源](tutorial-protect-resources.md)
