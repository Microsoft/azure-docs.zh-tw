---
title: Logic Apps 連接器概觀 | Microsoft Docs
description: 可在邏輯應用程式中使用的連接器概觀
services: ''
documentationcenter: ''
author: jeffhollan
manager: erikre
editor: ''
tags: connectors
ms.assetid: ca8dab2e-9b69-4b1e-865d-1facd9f0cdac
ms.service: logic-apps
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/15/2016
ms.author: jehollan
ms.openlocfilehash: 6f7d8f99bfa09847c01831a06efa8b94c1c0a89a
ms.sourcegitcommit: be9a42d7b321304d9a33786ed8e2b9b972a5977e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27961647"
---
# <a name="using-connectors-in-a-logic-app"></a>在邏輯應用程式中使用連接器
連接器可讓您跨服務、通訊協定與平台快速存取事件、資料和動作。  Logic Apps 所支援的連接器完整清單可以 [在這裡找到](apis-list.md)。  連接器可做為邏輯應用程式的觸發程序或動作，而且可能需要設定「連線」  才能使用 (例如︰授權 Twitter 帳戶代表您存取或張貼)。

## <a name="basics"></a>基本
連接器是邏輯應用程式中可供您存取以便整合 Dynamics、Azure、Salesforce [等等](apis-list.md)的其他服務的託管服務。  連接器是由 Microsoft 所部署和管理，因此您可以建置規模、輸送量和安全性都有人幫您處理的整合工作流程。  您可以藉由搜尋和選取 [顯示 Microsoft 受控 API] 底下的連接器動作或觸發程序，在邏輯應用程式中新增連接器。

![可供選取觸發程序的 [動作] 功能表][1]

每個連接器動作或觸發程序都有需要設定的屬性集。  您可以按一下 [資訊] 按鈕來深入了解動作，或參考其文件以 [深入了解](apis-list.md)。

如果您想要整合還未成為連接器的服務或 API，您也可以透過 [自訂連接器](../logic-apps/logic-apps-create-api-app.md) 擴充邏輯應用程式，或直接透過 HTTP 等通訊協定呼叫服務。

## <a name="triggers"></a>觸發程序
某些連接器具有觸發程序，這表示該連接器中的事件會引發邏輯應用程式，並傳入任何資料做為觸發程序的一部分。  觸發程序永遠是邏輯應用程式的第一個步驟。  受歡迎的觸發程序所包含的作業如下︰

* 循環 - 每小時執行一次
* 收到 HTTP 要求時
* 佇列中新增項目時
* 收到電子郵件時

在事件發生當下，有些觸發程序會在邏輯應用程式收到通知時立刻引發，有些則需要設定循環間隔，以指出邏輯應用程式檢查服務是否發生事件的頻率 (最多為每 15 秒)。  

一旦收到事件，就會引發執行邏輯應用程式，並開始工作流程中的動作。  您也可以在整個工作流程存取觸發程序的任何資料 (例如，「有新推文時」觸發程序會讓推文進入執行狀態)。

## <a name="actions"></a>動作
大部分連接器有一或多個可做為工作流程一部分來執行的動作。  動作是觸發程序已引發該次執行之後所發生的任何步驟。  若要新增動作，請按一下 [新增步驟]  按鈕，並搜尋您想要使用的連接器。  一經選取 (以及在設定任何可能需要的 [連線](#connections) 之後)，您就會看到可以設定的動作卡。  您可以按一下輸出的任何權杖以選取先前步驟的資料，或視需要輸入其他任何組態。

![設定連接器動作][2]

## <a name="connections"></a>連線
大部分連接器都必須先設定「連線」  然後才可供使用。  「連線」  是用來存取連接器所需的任何登入或連接組態。  對於使用 OAuth 的連接器，建立連線表示登入服務 (例如 Office 365、Salesforce 或 GitHub)，而您的存取權杖可加密並安全地儲存在 Azure 的密碼存放區。  其他連接器 (例如 FTP 和 SQL) 則需要包含伺服器位址、使用者名稱和密碼等組態的連線。  這些連線的組態詳細資料也會加密並安全地儲存。  只要在服務所允許的時間內，連線均可存取服務。  針對 Azure Active Directory OAuth 連線 (例如 Office 365 和 Dynamics)，我們可以無限期地不斷重新整理存取權杖。  其他服務則可能會限制權杖的使用時限，在此時限內，完全不需要重新整理權杖。  一般來說，某些動作 (例如變更密碼) 會導致所有存取權杖失效。  

按一下 [瀏覽]，然後選取 [API 連線]，即可在 Azure 中檢視和管理連線。  在 API 連線資源中，您可以檢視、編輯、更新或重新授權任何已建立的連線。

## <a name="next-steps"></a>後續步驟
* [建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [了解邏輯應用程式的常見用法和範例](../logic-apps/logic-apps-examples-and-scenarios.md)
* [開始使用企業整合觸發程序和動作](../logic-apps/logic-apps-enterprise-integration-overview.md)

<!--Image References -->
[1]: ./media/connectors-overview/addAction.png
[2]: ./media/connectors-overview/configureAction.png
