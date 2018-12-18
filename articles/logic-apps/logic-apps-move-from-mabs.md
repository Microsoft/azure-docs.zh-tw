---
title: 將應用程式從 BizTalk 服務移至 Azure Logic Apps | Microsoft Docs
description: 從 Azure BizTalk 服務 (MABS) 遷移至 Azure Logic Apps
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: jonfancey
ms.author: jonfan
ms.reviewer: estfan, LADocs
ms.topic: article
ms.date: 05/30/2017
ms.openlocfilehash: f27e82e780917e00625ef6a14ab8317d1f5b8ae8
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43124794"
---
# <a name="migrate-from-biztalk-services-to-azure-logic-apps"></a>從 BizTalk 服務遷移至 Azure Logic Apps

Microsoft Azure BizTalk 服務 (MABS) 即將停用。 若要將 MABS 整合解決方案移至 [Azure Logic Apps](../logic-apps/logic-apps-overview.md)，請遵循本文中的指引。 

## <a name="introduction"></a>簡介

BizTalk 服務包含兩個子服務：

* Microsoft BizTalk 服務混合式連線
* EAI 和 EDI 橋接器架構整合

[Azure App Service 混合式連線](../app-service/app-service-hybrid-connections.md)取代了 BizTalk 服務混合式連線。 透過 Azure 入口網站，即可使用 Azure App Service 取得 Azure 混合式連線。 此服務提供混合式連線管理員，以供您管理現有的 BizTalk 服務混合式連線，以及您在入口網站中建立的新混合式連線。 

[Logic Apps](../logic-apps/logic-apps-overview.md) 會以 BizTalk 服務等項目中的所有相同功能取代 EAI 和 EDI 橋接器型整合。 此服務提供以雲端規模使用情況為基礎的工作流程和協調流程功能，供您輕鬆快速地透過瀏覽器或 Visual Studio 建置複雜的整合解決方案。

此資料表會將 BizTalk 服務的功能對應至 Logic Apps。

| BizTalk 服務   | Logic Apps            | 目的                      |
| ------------------ | --------------------- | ---------------------------- |
| 連接器          | 連接器             | 傳送及接收資料   |
| 橋接器             | 邏輯應用程式             | 管線處理器           |
| 驗證階段     | XML 驗證動作 | 驗證 XML 文件的結構描述 | 
| 擴充階段       | 資料權杖           | 將屬性升級為訊息或針對路由決策升級屬性 |
| 轉換階段    | 轉換動作      | 將 XML 訊息從一種格式轉換成另一種格式 |
| 解碼階段       | 一般檔案解碼動作 | 從一般檔案轉換成 XML |
| 編碼階段       | 一般檔案編碼動作 | 從 XML 轉換成一般檔案 |
| 訊息檢查  | Azure Functions 或 API Apps | 在您的整合中執行自訂程式碼 |
| 路由動作       | 條件或 Switch | 將訊息路由傳送至其中一個指定的連接器 |
|||| 

## <a name="biztalk-services-artifacts"></a>BizTalk 服務構件

BizTalk 服務有數種構件。 

## <a name="connectors"></a>連接器

BizTalk 服務連接器可協助橋接器傳送及接收資料，包括啟用 HTTP 要求/回應互動的雙向橋接器。 Logic Apps 使用相同的術語，並且具有 180 個以上的連接器，這些連接器會藉由連線至許多技術和服務來起到相同作用。 舉例來說，連接器可用於雲端 SaaS 和 PaaS 服務 (例如 OneDrive、Office365、Dynamics CRM 等等)，並可透過內部部署資料閘道 (其取代了 BizTalk 服務的 BizTalk 配接器服務) 在內部部署系統中使用。 BizTalk 服務中的來源限制在 FTP、SFTP 和服務匯流排佇列或主題訂閱。

![](media/logic-apps-move-from-mabs/sources.png)

根據預設，每個橋接器預設有一個 HTTP 端點，並以橋接器的「執行階段位址」和「相對位址」屬性設定。 若要使用 Logic Apps 達成相同結果，請使用[要求和回應](../connectors/connectors-native-reqres.md)動作。

## <a name="xml-processing-and-bridges"></a>XML 處理和橋接器

在 BizTalk 服務中，橋接器類似於處理管線。 橋接器可以擷取從連接器收到的資料，使用該資料執行一些工作，再將結果傳送至另一個系統。 Logic Apps 會藉由支援與 BizTalk 服務相同的管線互動模式來執行相同的作業，此外還提供其他整合模式。 BizTalk 服務中的 [XML 要求-回覆橋接器](https://msdn.microsoft.com/library/azure/hh689781.aspx)也稱為 VETER 管線，組成此管線的階段可以執行以下工作：

* (V) 驗證
* (E) 擴充
* (T) 轉換
* (E) 擴充
* (R) 路由

下圖顯示要求與回覆之間會如何分割處理，以分開控制要求與回覆路徑 (例如，藉由對每個路徑使用不同對應)：

![](media/logic-apps-move-from-mabs/xml-request-reply.png)

此外，XML 單向橋接器會在處理開頭和結尾新增解碼和編碼階段。 傳遞橋接器則包含單一擴充階段。

### <a name="message-processing-decoding-and-encoding"></a>訊息處理、解碼和編碼

在 BizTalk 服務中，您可以接收不同類型的 XML 訊息，並判斷所收到訊息的結構描述是否相符。 這項工作會在接收處理管線的「訊息類型」階段中執行。 然後，解碼階段會使用偵測到的訊息類型，並搭配提供的結構描述來將訊息解碼。 如果結構描述是一般檔案結構描述，這個階段會將傳入的一般檔案轉換成 XML。 

Logic Apps 提供類似的功能。 您可以使用不同的連接器觸發程序 (檔案系統、FTP、HTTP 等)，透過不同的通訊協定來接收一般檔案，然後使用[一般檔案解碼](../logic-apps/logic-apps-enterprise-integration-flatfile.md)動作將傳入資料轉換成 XML。 您可以將現有的一般檔案結構描述直接移至 Logic Apps，而不需要進行任何變更，再將結構描述上傳至您的整合帳戶。

### <a name="validation"></a>驗證

將傳入資料轉換成 XML 之後 (或如果 XML 是收到的訊息格式)，執行驗證來判斷訊息是否符合您的 XSD 結構描述。 若要在 Logic Apps 中執行這項工作，請使用 [XML 驗證](../logic-apps/logic-apps-enterprise-integration-xml-validation.md)動作。 您可以從 BizTalk 服務使用相同的結構描述，而不需要進行任何變更。

### <a name="transform-messages"></a>轉換訊息

在 BizTalk 服務中，轉換階段會將一種 XML 訊息格式轉換成另一種格式。 這項工作會藉由使用 TRFM 對應工具套用對應來完成。 在 Logic Apps 中，此程序會很類似。 轉換動作會從您的整合帳戶執行對應。 主要差異在於 Logic Apps 中的對應採用 XSLT 格式。 XSLT 可以重複使用您目前已有的 XSLT，包括針對 BizTalk Server 所建立且含有運算質的對應。 

### <a name="routing-rules"></a>路由規則

BizTalk 服務會做出由哪一個端點或連接器來傳送傳入訊息或資料的路由決策。 使用路由篩選選項，即可從預先設定的端點中選取：

![](media/logic-apps-move-from-mabs/route-filter.png)

在 BizTalk 服務中，如果只有兩個選項，使用「條件」會是在 BizTalk 服務中轉換路由篩選的最佳方法。 如果有兩個以上的選項，則使用 **switch**。

Logic Apps 提供複雜的邏輯功能，以及具有[條件式陳述式](../logic-apps/logic-apps-control-flow-conditional-statement.md)和[參數陳述式](../logic-apps/logic-apps-control-flow-switch-statement.md)的進階控制流程和路由。

### <a name="enrich"></a>擴充

在 BizTalk 服務處理中，擴充階段會將屬性新增至與所接收資料相關聯的訊息內容。 例如，升級屬性以用於從資料庫尋查路由，或藉由使用 XPath 運算式來擷取值。 Logic Apps 可從上述動作存取所有關聯式資料輸出，因此您可以直接複寫相同的行為。 例如，使用 `Get Row` SQL 連線動作，您可以從 SQL Server 資料庫傳回資料，並在決策動作中使用資料進行路由。 同樣地，觸發程序傳入的服務匯流排佇列訊息的屬性是可定址的，而使用 xpath 工作流程定義語言運算式的 XPath 也是。

### <a name="run-custom-code"></a>執行自訂程式碼

BizTalk 服務讓您能夠在自己的組件中[執行上傳的自訂程式碼](https://msdn.microsoft.com/library/azure/dn232389.aspx)。 此功能會由 [IMessageInspector](https://msdn.microsoft.com/library/microsoft.biztalk.services.imessageinspector) 介面來實作。 橋接器中的每個階段包含兩個屬性 (On Enter Inspector 和 On Exit Inspector)，提供您建立來實作此介面的 .NET 類型。 自訂程式碼可讓您對資料執行更複雜的處理，以及在執行通用商務邏輯的組件中重複使用現有的程式碼。 

Logic Apps 提供兩個執行自訂程式碼的主要方法：Azure Functions 和 API Apps。 您可以建立 Azure Functions，並從 Logic Apps 呼叫。 請參閱[透過 Azure Functions 新增並執行適用於 Logic Apps 的自訂程式碼](../logic-apps/logic-apps-azure-functions.md)。 使用 Azure App Service 一部分的 API Apps，可建立您自己的觸發程序和動作。 深入了解[建立自訂 API 來與 Logic Apps 搭配使用](../logic-apps/logic-apps-create-api-app.md)。 

如果您的組件中有從 BizTalk 服務呼叫的自訂程式碼，您可以將此程式碼移至 Azure Functions，或使用 API Apps 建立自訂 API，視您實作的項目而定。 例如，如果您有包裝其他服務的程式碼，且 Logic Apps 沒有該服務的連接器，請建立 API 應用程式，並使用 API 應用程式在邏輯應用程式內提供的動作。 如果您有 Helper 函式或程式庫，則 Azure Functions 可能是最好的選擇。

### <a name="edi-processing-and-trading-partner-management"></a>EDI 處理和交易夥伴管理

BizTalk 服務和 Logic Apps 包含 EDI 和 B2B 處理，並支援 AS2 (Applicability Statement 2)、X12 和 EDIFACT。 在 BizTalk 服務中，您可以建立 EDI 橋接器，並在專用追蹤與管理入口網站中建立或管理交易夥伴和協議。
在 Logic Apps 中，您可以透過[企業整合套件 (EIP)](../logic-apps/logic-apps-enterprise-integration-overview.md) 取得這項功能。 EIP 提供[整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)和適用於 EDI 與 B2B 處理的 B2B 動作。 您也可以使用整合帳戶來建立及管理[交易夥伴](../logic-apps/logic-apps-enterprise-integration-partners.md)和[協議](../logic-apps/logic-apps-enterprise-integration-agreements.md)。 建立整合帳戶之後，您可以將一或多個邏輯應用程式連結至該帳戶。 然後，您可以使用 B2B 動作從邏輯應用程式存取合作對象的資訊。 提供下列動作：

* AS2 編碼
* AS2 解碼
* X12 編碼
* X12 解碼
* EDIFACT 編碼
* EDIFACT 解碼

不同於 BizTalk 服務，這些動作會從傳輸通訊協定分離。 因此當您建立 Logic Apps 時，您有更大的彈性來選擇要用來傳送及接收資料的連接器。 例如，您可以接收電子郵件附件形式的 X12 檔案，然後在邏輯應用程式中處理這些檔案。 

## <a name="manage-and-monitor"></a>管理和監視

在 BizTalk 服務中，專用入口網站提供了追蹤功能供您監視問題並進行疑難排解。 Logic Apps 透過 [Azure 入口網站](../logic-apps/logic-apps-monitor-your-logic-apps.md)提供更豐富的追蹤和監視功能，包括可在移動時留意各項事物的行動裝置應用程式。

## <a name="high-availability"></a>高可用性

若要在 BizTalk 服務中達到高可用性 (HA)，您可以在特定區域中使用多個執行個體來分攤處理負載。 Logic Apps 提供區域中 HA，且不需額外成本。 

在 BizTalk 服務中，B2B 處理的區域外災害復原需要備份和還原程序。 針對商務持續性，Logic Apps 提供跨區域主動/被動 [DR 功能](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md)，可讓您在不同區域的整合帳戶之間同步處理 B2B 資料。

## <a name="next-steps"></a>後續步驟

* [Logic Apps 是什麼？](../logic-apps/logic-apps-overview.md)
* [建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)，或使用[預先建置的範本](../logic-apps/logic-apps-create-logic-apps-from-templates.md)來快速上手  
* [檢視所有可用的連接器](../connectors/apis-list.md)，您可以在邏輯應用程式中使用這些連接器