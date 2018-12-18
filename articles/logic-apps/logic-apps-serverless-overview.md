---
title: Azure 無伺服器概觀 | Microsoft Docs
description: 了解如何在雲端建立強大的解決方案，而不必擔心基礎結構
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: jeffhollan
ms.author: jehollan
ms.reviewer: klam, estfan, LADocs
ms.custom: vs-azure
ms.topic: article
ms.date: 03/30/2017
ms.openlocfilehash: 97c928c34a18a5d4f3549c348a273df268ee1db0
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123303"
---
# <a name="overview-azure-serverless-with-azure-logic-apps-and-azure-functions"></a>概觀：Azure 無伺服器搭配 Azure Logic Apps 和 Azure Functions

無伺服器應用程式提供的好處包括加速開發、減少必要的程式碼，以及簡單調整。  本文描述無伺服器解決方案的各種屬性和 Azure 無伺服器供應項目。

## <a name="what-is-serverless"></a>什麼是無伺服器？

無伺服器並不表示沒有伺服器 - 只是表示開發人員不需要擔心伺服器。  大部分的傳統應用程式開發主要是回答有關調整、裝載及監視解決方案的問題，以滿足應用程式的需求。  在無伺服器中，這些問題由解決方案本身來處理。  此外，無伺服器應用程式是以耗用量為基礎的方案計費。  如果從未使用過應用程式，則永遠不會產生費用。  這些功能可讓開發人員專注於解決方案的商務邏輯。

Azure 中有關非伺服器的核心服務是 [Azure Functions](https://azure.microsoft.com/services/functions/) 和 [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/)。  這兩種解決方案遵循上述原則，可讓開發人員以最少程式碼建置健全的雲端應用程式。

## <a name="what-are-azure-functions"></a>什麼是 Azure Functions？

Azure Functions 是可在雲端輕鬆執行程式碼片段或「函數」的解決方案。 您可以只撰寫處理手邊問題所需的程式碼，而不需擔心要執行它的整個應用程式或基礎結構。 Functions 可讓開發更有生產力，而且您可以使用您選擇的開發語言，例如 C#、F#、Node.js、Python 或 PHP。 只需依程式碼執行的時間多寡付費，Azure 會視需要而調整。

如果您想要直接進入正題並開始使用 Azure Functions，請從 [建立您的第一個Azure Functions](../azure-functions/functions-create-first-azure-function.md)著手。 如果您要尋找更多有關 Functions 的技術資訊，請參閱 [開發人員參考](../azure-functions/functions-reference.md)。

## <a name="what-are-azure-logic-apps"></a>什麼是 Azure Logic Apps？

Azure Logic Apps 能夠在雲端中簡化和實作可調整的整合和工作流程。 它提供視覺化設計工具，以一系列的步驟 (稱為工作流程) 為您的程序建立模型並加以自動化。  雲端和內部部署服務中有[許多連接器](../connectors/apis-list.md)，可快速將無伺服器應用程式連線至其他 API。  邏輯應用程式是以觸發程序為開端 (如「當帳戶加入至 Dynamics CRM 時」)，而在觸發後可以開始處理各種組合的動作、轉換和條件邏輯。  在一個程序中協調不同的 Azure Functions 時，Logic Apps 是絕佳選擇 - 尤其是當程序需要與外部系統或 API 互動時。

若要開始使用 Logic Apps，請從[建立您的第一個邏輯應用程式](quickstart-create-first-logic-app-workflow.md)開始。  如果您要尋找更多有關 Logic Apps 的技術資訊，請參閱[開發人員參考](logic-apps-workflow-actions-triggers.md)。

## <a name="how-can-i-build-and-deploy-serverless-applications-in-azure"></a>如何在 Azure 中建置和部署無伺服器應用程式？

Azure 提供一組豐富的工具，涵蓋開發、部署和管理無伺服器應用程式。  您可以直接在 Azure 入口網站中建置應用程式，或使用 [Visual Studio 中的工具](logic-apps-serverless-get-started-vs.md)來建置。  開發應用程式之後，就可以[立即部署](logic-apps-create-deploy-template.md)。  Azure 也支援監視無伺服器應用程式。  您可以從 Azure 入口網站、透過 API 或 SDK，或使用 Log Analytics 和 Application Insights 的整合式工具，存取此監視功能。

## <a name="next-steps"></a>後續步驟

* [開始在 Visual Studio 中建置無伺服器應用程式](logic-apps-serverless-get-started-vs.md)
* [使用無伺服器建立 Customer Insights 儀表板](logic-apps-scenario-social-serverless.md)
* [建置邏輯應用程式的部署範本](logic-apps-create-deploy-template.md)