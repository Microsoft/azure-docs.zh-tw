---
title: 定價 & 計費 - Azure Logic Apps | Microsoft Docs
description: 了解 Azure Logic Apps 的定價和計費方式
services: logic-apps
ms.service: logic-apps
ms.suite: logic-apps
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, LADocs
ms.assetid: f8f528f5-51c5-4006-b571-54ef74532f32
ms.topic: article
ms.date: 09/24/2018
ms.openlocfilehash: 5f9147035c07bbe4fb3f38b74025015e70dd87b3
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47159548"
---
# <a name="pricing-model-for-azure-logic-apps"></a>Azure Logic Apps 的定價模式

您可以使用 Azure Logic Apps，在雲端建立和執行可調整規模的自動化整合工作流程。 以下是關於 Logic Apps 計費和定價運作方式的詳細資料。 

<a name="consumption-pricing"></a>

## <a name="consumption-pricing-model"></a>取用量定價模式

對於使用公用或「全域」Logic Apps 服務建立的新邏輯應用程式，您只需針對您使用的部分付費。 這些邏輯應用程式會使用取用量方案和定價模型，這表示由邏輯應用程式所執行的所有動作執行都會納入計量。 邏輯應用程式定義中的每個步驟都是一個動作，包括觸發程序、控制流程步驟、呼叫內建動作，以及呼叫連接器。 如需詳細資訊，請參閱 [Logic Apps 價格](https://azure.microsoft.com/pricing/details/logic-apps)。

<a name="fixed-pricing"></a>

## <a name="fixed-pricing-model"></a>固定定價模式

> [!NOTE]
> 整合服務環境目前為*個人預覽版*。 若要要求存取，請在[這裡建立您的加入要求](https://aka.ms/iseprivatepreview)。

您透過[*整合服務環境* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) 建立的新邏輯應用程式是隔離的私人 Logic Apps 執行個體，使用專用的資源，對於這個新的邏輯應用程式，您每個月會支付固定的費用來使用其內建的動作與標準 ISE 標示的連接器。 您的 ISE 免費隨附一個企業連接器，如需額外的企業連接器，則會根據企業使用量價格計費。 如需詳細資訊，請參閱 [Logic Apps 價格](https://azure.microsoft.com/pricing/details/logic-apps)。

<a name="triggers"></a>

## <a name="triggers"></a>觸發程序

觸發程序是在特定事件發生時建立邏輯應用程式執行個體的特殊動作。 觸發程序會以不同動作影響邏輯應用程式計量方式。

* **輪詢觸發程序**：此觸發程序會持續檢查端點來取得訊息，其會符合符合用來建立邏輯應用程式執行個體以啟動工作流程的準則。 每個輪詢要求都會以一個執行來計算，並進行計量，即使未建立任何邏輯應用程式執行個體時也一樣。 您可以透過邏輯應用程式設計工具來設定觸發程序，以指定輪詢間隔。

  [!INCLUDE [logic-apps-polling-trigger-non-standard-metering](../../includes/logic-apps-polling-trigger-non-standard-metering.md)]

* **Webhook 觸發程序**：此觸發程序會等候用戶端將要求傳送到特定端點。 每個傳送至 Webhook 端點的要求都會計算為一個動作執行。 例如，要求和 HTTP Webhook 觸發程序都是 Webhook 觸發程序。

* **週期性觸發程序**：此觸發程序會根據您在觸發程序中設定的週期間隔來建立邏輯應用程式執行個體。 例如，您可以設定一個週期性觸發程序，每隔三天執行一次或基於更複雜的排程來執行。

您可以在邏輯應用程式的 [概觀] 窗格中，於 [觸發程序歷程記錄] 區段下方找到觸發程序執行。

## <a name="actions"></a>動作

內建動作 (例如，呼叫 HTTP、Azure Functions 或 API 管理的動作，以及控制流程步驟) 會以原生動作來計量，而且其具有其各自的類型。 呼叫[連接器](https://docs.microsoft.com/connectors) \(英文\) 的動作具有 "ApiConnection" 類型。 這些連接器會歸類為標準或企業連接器，並以其各自的[定價][pricing]來計量。 預覽版的企業連接器會依標準連接器的標準收費。

所有成功和失敗的執行動作都會納入計算，並以動作執行來計量。 不過，因為條件不符而略過的動作，或因為邏輯應用程式在完成之前就終止而未執行的動作，均不會當成動作執行來計算。 停用的邏輯應用程式無法將新的執行個體具現化，因此，停用它們時，就不會針對它們進行收費。

> [!NOTE]
> 停用邏輯應用程式之後，其目前正在執行的執行個體可能需要一些時間才能完全停止。

在迴圈內部執行的動作會針對迴圈中的每個週期來計算。 例如，在 "for each" 迴圈中處理 10 個項目之清單的單一動作會以下列方式來計算：將清單項目數 (10) 乘以迴圈中的動作數 (1)，再加上用來啟動迴圈的一。 因此，針對此範例的計算是 (10 * 1) + 1，結果就是 11 個動作執行。

## <a name="integration-account-usage"></a>整合帳戶使用量

以取用量為根據的使用量包括一個[整合帳戶](logic-apps-enterprise-integration-create-integration-account.md)，您可以在其中探勘、開發和測試 Logic Apps 的 [B2B/EDI](logic-apps-enterprise-integration-b2b.md) 和 [XML 處理](logic-apps-enterprise-integration-xml.md)功能，而不需額外成本。 您可以在每個區域擁有一個整合帳戶，並可儲存一定[成品數量](../logic-apps/logic-apps-limits-and-config.md)，例如，EDI 交易夥伴和協議、對應、結構描述、組件、憑證和批次組態。

Logic Apps 也會搭配支援的 Logic Apps SLA 來提供基本和標準整合帳戶。 當您只想使用訊息處理，或者做為與較大商務實體有交易夥伴關係的小型企業夥伴時，就可以使用基本整合帳戶。 標準整合帳戶支援更複雜的 B2B 關係，並增加您可以管理的實體數目。 如需詳細資訊，請參閱 [Azure 價格](https://azure.microsoft.com/pricing/details/logic-apps)。

## <a name="next-steps"></a>後續步驟

* [深入了解 Logic Apps][whatis]
* [建立第一個邏輯應用程式][create]

[pricing]: https://azure.microsoft.com/pricing/details/logic-apps/
[whatis]: logic-apps-overview.md
[create]: quickstart-create-first-logic-app-workflow.md

