---
title: 使用成本警示監視使用量和支出
description: 本文說明成本警示如何協助您在 Azure 成本管理中監視使用量和支出。
author: bandersmsft
ms.author: banders
ms.date: 09/03/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.openlocfilehash: d1546ef92d923b9d72c4aca0ddf61d2b93646656
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98602340"
---
# <a name="use-cost-alerts-to-monitor-usage-and-spending"></a>使用成本警示監視使用量和支出

本文將協助您了解並使用成本管理警示，來監視您的 Azure 使用量和支出。 成本警示會根據取用 Azure 資源的時機自動產生。 警示會在同一個地方顯示所有作用中的成本管理和帳單警示。 當您的使用量到達指定的臨界值時，成本管理就會產生警示。 有三種類型的成本警示：預算警示、點數警示，以及部門支出配額警示。

## <a name="budget-alerts"></a>預算警示

預算警示會在支出 (根據使用量或成本) 到達或超過[預算的警示條件](tutorial-acm-create-budgets.md)中所定義的數量時通知您。 成本管理預算會使用 Azure 入口網站或 [Azure 使用量](/rest/api/consumption) \(英文\) API 來建立。

在 Azure 入口網站中，預算會依成本來定義。 使用 Azure 使用量 API，預算會依成本或使用量來定義。 預算警示支援以成本為基礎和以使用量為基礎的預算。 每當預算警示條件相符時，即會自動產生預算警示。 您可以在 Azure 入口網站中檢視所有成本警示。 每當警示產生時，就會顯示於成本警示中。 系統也會傳送警示電子郵件給預算警示收件者清單中的人員。

您可以使用預算 API 以不同的語言傳送電子郵件警示。 如需詳細資訊，請參閱[預算警示電子郵件支援的地區設定](manage-automation.md#supported-locales-for-budget-alert-emails)。

## <a name="credit-alerts"></a>點數警示

當您的 Azure 預付款 (先前稱為預付金) 被取用時，信用額度警示會通知您。 Azure 預付款適用於具有 Enterprise 合約的組織。 信用額度警示會在您 Azure 預付款信用額度餘額的 90% 和 100% 時自動產生。 每當警示產生時，就會在成本警示和傳送給帳戶擁有者的電子郵件中反映該警示。

## <a name="department-spending-quota-alerts"></a>部門費用配額警示

部門費用配額警示會在部門費用達到配額的固定閾值時通知您。 費用配額是在 EA 入口網站中設定。 每當達到閾值時，此警示就會產生一封電子郵件傳送給部門擁有者，並顯示在成本警示中。 例如，配額的 50% 或 75%。

## <a name="supported-alert-features-by-offer-categories"></a>依供應項目類別支援的警示功能

警示類型的支援取決於您擁有的 Azure 帳戶類型 (Microsoft 供應項目)。 下表顯示各種 Microsoft 供應項目所支援的警示功能。 您可以在[了解成本管理資料](understand-cost-mgt-data.md)中檢視 Microsoft 供應項目的完整清單。

| 警示類型 | Enterprise 合約 | Microsoft 客戶合約 | Web Direct/隨用隨付 |
|---|---|---|---|
| 預算 | ✔ | ✔ | ✔ |
| 點數 | ✔ |✘ | ✘ |
| 部門支出配額 | ✔ | ✘ | ✘ |



## <a name="view-cost-alerts"></a>檢視成本警示

若要檢視預算警示，請在 Azure 入口網站中開啟所需的範圍，然後選取功能表中的 [預算]。 使用 [範圍] 膠囊切換到不同的範圍。 選取功能表中的 [成本警示]。 如需有關範圍的詳細資訊，請參閱[了解並使用範圍](understand-work-scopes.md)。

![在成本管理中顯示的警示範例影像](./media/cost-mgt-alerts-monitor-usage-spending/budget-alerts-fullscreen.png)

作用中和已關閉的警示總數會顯示在成本警示頁面上。

所有警示都會顯示警示類型。 預算警示會顯示它的產生原因及其適用的預算名稱。 每個警示都會顯示它的產生日期、它的狀態，以及警示適用的範圍 (訂用帳戶或管理群組)。

可能的狀態包括 **作用中** 和 **已關閉**。 [作用中] 狀態表示警示仍然相關。 [已關閉] 狀態表示有人標示了警示，將其設定為不再相關。

從清單中選取要檢視其詳細資料的警示。 警示詳細資料會顯示警示的詳細資訊。 預算警示包含預算的連結。 如果有適用於預算警示的建議，則也會顯示建議的連結。 預算、點數和部門支出配額警示都會提供連結以便在成本分析中進行分析，而您可以在其中瀏覽警示範圍的成本。 下列範例顯示某部門的支出與警示詳細資料。

![顯示某部門的支出與警示詳細資料的範例影像](./media/cost-mgt-alerts-monitor-usage-spending/dept-spending-selected-with-credits.png)

當您檢視已關閉警示的詳細資料時，如果需要手動動作，則您可以重新啟動它。 下圖說明一個範例。

![顯示關閉並重新啟動選項的範例影像](./media/cost-mgt-alerts-monitor-usage-spending/Dismiss-reactivate-options.png)

## <a name="see-also"></a>另請參閱

- 如果您尚未建立預算或設定預算的警示條件，請完成[建立和管理預算](tutorial-acm-create-budgets.md)教學課程。