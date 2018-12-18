---
title: Azure 資訊安全中心的安全性劇本 | Microsoft Docs
description: 這份文件可協助您使用 Azure 資訊安全中心的安全性劇本，以自動化您對於安全性事件的回應。
services: security-center
documentationcenter: na
author: terrylan
manager: mbaldwin
editor: ''
ms.assetid: a8c45ddf-5c4c-4393-b6e9-46ed1f91bf5f
ms.service: security-center
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/09/2018
ms.author: yurid
ms.openlocfilehash: 1d6fee9cb46fcfec32c0ecdb91eec09a56063c4e
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32779272"
---
# <a name="security-playbook-in-azure-security-center-preview"></a>Azure 資訊安全中心的安全性劇本 (預覽)
這份文件可協助您使用 Azure 資訊安全中心的安全性劇本，以回應安全性相關問題。

## <a name="what-is-security-playbook-in-security-center"></a>資訊安全中心的安全性劇本是什麼？
安全性劇本是程序的集合，一旦特定劇本從選取的警示觸發，則可以從資訊安全中心執行這些程序。 安全性劇本可協助自動化以及協調您對於資訊安全中心偵測到之特定安全性警示的回應。 資訊安全中心的安全性劇本是根據 [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps)，這表示您可以使用在 Logic Apps 範本中安全性類別底下所提供的範本，您可以根據您的需求進行修改，或者可以使用 [Azure Logic Apps 工作流程](https://docs.microsoft.com/azure/logic-apps/logic-apps-create-a-logic-app)建立新的劇本，並且使用資訊安全中心作為觸發程序。 

> [!NOTE]
> 劇本會運用 Azure Logic Apps，因此會產生費用。 請造訪 [Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/) 價格分頁以取得詳細資料。 

## <a name="how-to-create-a-security-playbook-from-security-center"></a>如何從資訊安全中心建立安全性劇本？
遵循以下步驟以從資訊安全中心建立新的安全性劇本：

1.  開啟 [資訊安全中心] 儀表板。
2.  在左窗格的 [自動化與協調流程] 區段中，按一下 [劇本 (預覽)]。

    ![邏輯應用程式](./media/security-center-playbooks/security-center-playbooks-fig17.png)
 
3. 在 [資訊安全中心 - 劇本 (預覽)] 分頁中，按一下 [新增] 按鈕。

    ![建立邏輯應用程式](./media/security-center-playbooks/security-center-playbooks-fig2.png)

4. 在 [建立邏輯應用程式] 分頁上，輸入所要求的資訊以建立新的邏輯應用程式，然後按一下 [建立] 按鈕。 一旦建立完成，新的劇本會出現在清單中。 如果它沒有出現，請按一下 [重新整理] 按鈕。 一旦您看到它，請按一下它以開始編輯這個劇本。

    ![建立邏輯應用程式](./media/security-center-playbooks/security-center-playbooks-fig3.png)

5. [邏輯應用程式設計工具] 隨即出現。 按一下 [空白邏輯應用程式] 以建立新的劇本。 您也可以選取類別底下的 [安全性]，然後使用其中一個範本。
    
    ![邏輯應用程式設計工具](./media/security-center-playbooks/security-center-playbooks-fig4.png)

6. 在 [搜尋所有連接器及觸發程序] 欄位中，輸入 [Azure 資訊安全中心]，然後選取 [當對 Azure 資訊安全中心警示的回應已觸發時]。

    ![觸發程序](./media/security-center-playbooks/security-center-playbooks-fig12.png)

7. 現在您可以定義當觸發劇本時會發生什麼事。 您可以新增動作、邏輯條件、切換案例條件或迴圈。

    ![邏輯應用程式設計工具](./media/security-center-playbooks/security-center-playbooks-fig5.png)
     
## <a name="how-to-run-a-security-playbook-in-security-center"></a>如何執行資訊安全中心的安全性劇本？

當您想要協調、從其他服務取得詳細資訊，或當您想要修復時，您可以執行資訊安全中心的安全性劇本。 若要存取劇本，請遵循下列步驟：

1.  開啟 [資訊安全中心] 儀表板。
2.  在左窗格中的 [威脅偵測] 底下，按一下 [安全性事件與警示]。

    ![警示](./media/security-center-playbooks/security-center-playbooks-fig6.png)

3.  按一下您想要調查的警示。
4.  在警示分頁頂端，按一下 [執行劇本] 按鈕。

    ![執行劇本](./media/security-center-playbooks/security-center-playbooks-fig7.png)

5. 在 [劇本] 分頁中，選取您想要執行的劇本，然後按一下 [執行] 按鈕。 如果您想要在觸發之前查看劇本，您可以按一下它，設計工具隨即開啟。

    ![劇本](./media/security-center-playbooks/security-center-playbooks-fig13.png)

### <a name="history"></a>歷程記錄

執行劇本之後，您也可以存取先前的執行，以及包含先前執行劇本之狀態相關詳細資訊的步驟。 記錄在每個劇本中都是內容相關的，這表示您在此分頁中看到的劇本記錄是與觸發此劇本的警示相互關聯。 

![歷程記錄](./media/security-center-playbooks/security-center-playbooks-fig16.png)

若要查看特定劇本執行的詳細資料，請按一下劇本本身，具有整個工作流程的邏輯應用程式執行分頁隨即顯示。

![詳細資料](./media/security-center-playbooks/security-center-playbooks-fig14.png)

在此工作流程中，您可以看到每個工作執行的時間，而且您可以展開每個工作以查看結果。 

### <a name="changing-an-existing-playbook"></a>變更現有劇本

您可以變更資訊安全中心的現有劇本，以新增動作或條件。 若要這樣做，您只需要在 [劇本] 索引標籤中按一下您想要變更之劇本的名稱，[邏輯應用程式設計工具] 隨即開啟。

> [!NOTE]
> 如需如何使用 Azure Logic App 來建立自己的劇本的詳細資訊，請參閱[建立第一個邏輯應用程式工作流程來自動執行雲端應用程式與雲端服務之間的程序](https://docs.microsoft.com/azure/logic-apps/logic-apps-create-a-logic-app#add-an-action-that-responds-to-your-trigger)。


## <a name="see-also"></a>另請參閱
在本文件中，您已了解如何在 Azure 資訊安全中心內使用劇本。 若要深入了解「Azure 資訊安全中心」，請參閱下列主題：

* [管理及回應 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)。 了解如何在資訊安全中心管理警示，以及回應安全性事件。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md)。 了解如何監視 Azure 資源的健全狀況。
* [了解 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)。 了解不同類型的安全性警示。
* [Azure 資訊安全中心疑難排解指南](https://docs.microsoft.com/azure/security-center/security-center-troubleshooting-guide)。 了解如何針對資訊安全中心的常見問題進行疑難排解。 
* [Azure 資訊安全中心常見問題集](security-center-faq.md)。 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)。 尋找有關 Azure 安全性與相容性的部落格文章。

