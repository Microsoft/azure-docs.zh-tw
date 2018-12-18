---
title: 使用 Liquid 轉換來轉換 JSON 資料 - Azure Logic Apps | Microsoft Docs
description: 使用 Logic Apps 和 Liquid 範本建立進階 JSON 轉換的轉換或對應
services: logic-apps
ms.service: logic-apps
author: divyaswarnkar
ms.author: divswa
manager: jeconnoc
ms.reviewer: estfan, LADocs
ms.suite: integration
ms.topic: article
ms.date: 08/16/2018
ms.openlocfilehash: 140c92d260ac6423127e478e304cbebcf9c42124
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42144812"
---
# <a name="perform-advanced-json-transformations-with-liquid-templates-in-azure-logic-apps"></a>使用 Azure Logic Apps 中的 Liquid 範本執行進階的 JSON 轉換

您可以透過原生資料作業動作 (例如**撰寫**或**剖析 JSON**)，在邏輯應用程式中進行基本 JSON 轉換。 若要執行進階的 JSON 轉換，您可以使用 [Liquid](https://shopify.github.io/liquid/) 建立範本或對應，Liquid 是適用於可變動 Web 應用程式的開放原始碼範本語言。 Liquid 範本可讓您定義轉換 JSON 輸出的方式，並可支援更複雜的 JSON 轉換，例如反覆項目、控制流程、變數等等。 

因此，您在邏輯應用程式中執行 Liquid 轉換之前，可先使用 Liquid 範本來定義 JSON 到 JSON 的對應，並在您的企業整合帳戶中儲存該對應。 本文會示範如何建立及使用此 Liquid 範本或對應。 

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有訂用帳戶，您可以[開始使用免費 Azure 帳戶](https://azure.microsoft.com/free/)。 或者，請[註冊預付型方案訂用帳戶](https://azure.microsoft.com/pricing/purchase-options/)。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 基本[企業整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)

## <a name="create-liquid-template-or-map-for-your-integration-account"></a>為您的企業整合帳戶建立 Liquid 範本或對應

1. 在此範例中，您會建立此步驟中描述的 Liquid 範本範例。
如果您想要在 Liquid 範本中使用任何篩選，請確定這些篩選以大寫字母開頭。 深入了解 [Liquid 篩選](https://shopify.github.io/liquid/basics/introduction/#filters)。 

   ```json
   {%- assign deviceList = content.devices | Split: ', ' -%}
   
   {
      "fullName": "{{content.firstName | Append: ' ' | Append: content.lastName}}",
      "firstNameUpperCase": "{{content.firstName | Upcase}}",
      "phoneAreaCode": "{{content.phone | Slice: 1, 3}}",
      "devices" : [
         {%- for device in deviceList -%}
            {%- if forloop.Last == true -%}
            "{{device}}"
            {%- else -%}
            "{{device}}",
            {%- endif -%}
        {%- endfor -%}
        ]
   }
   ```

2. 登入 [Azure 入口網站](https://portal.azure.com)。 在主要 Azure 功能表上，選取 [所有資源]。 在搜尋方塊中，尋找並選取整合帳戶。

   ![選取企業整合帳戶](./media/logic-apps-enterprise-integration-liquid-transform/select-integration-account.png)

3.  在 [元件] 底下，選取 [對應]。

    ![選取對應](./media/logic-apps-enterprise-integration-liquid-transform/add-maps.png)

4. 選擇 [新增]，並提供該對應的以下詳細資料：

   | 屬性 | 值 | 說明 | 
   |----------|-------|-------------|
   | **名稱** | JsonToJsonTemplate | 對應的名稱，在此範例中是 "JsonToJsonTemplate" | 
   | **對應類型** | **liquid** | 對應的類型。 對於 JSON 到 JSON 的轉換，請選取 [Liquid]。 | 
   | **地圖** | "SimpleJsonToJsonTemplate.liquid" | 用於轉換的現有 Liquid 範本或對應檔案，在此範例中是 "SimpleJsonToJsonTemplate.liquid"。 若要尋找此檔案，您可以使用檔案選擇器。 |
   ||| 

   ![新增 Liquid 範本](./media/logic-apps-enterprise-integration-liquid-transform/add-liquid-template.png)
    
## <a name="add-the-liquid-action-for-json-transformation"></a>新增 JSON 轉換的 Liquid 動作

1. 在 Azure 入口網站中，遵循下列步驟來[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

2. 在邏輯應用程式設計工具中，將[要求觸發程序](../connectors/connectors-native-reqres.md#use-the-http-request-trigger)新增至邏輯應用程式。

3. 在觸發程序下方，選擇 [新增步驟]。 在 [搜尋] 方塊中，輸入 "liquid" 作為篩選條件，並選取下列動作：**將 JSON 轉換為 JSON - Liquid**

   ![尋找並選取 Liquid 動作](./media/logic-apps-enterprise-integration-liquid-transform/search-action-liquid.png)

4. 在 [內容] 方塊內按一下，動態內容清單隨即顯示，然後選取 [內文] 權杖。
  
   ![選取內文](./media/logic-apps-enterprise-integration-liquid-transform/select-body.png)
 
5. 從 [對應] 清單中，選取您的 Liquid 範本，在此範例中是「JsonToJsonTemplate」。

   ![選取對應](./media/logic-apps-enterprise-integration-liquid-transform/select-map.png)

   如果清單是空的，可能表示邏輯應用程式未連結至您的企業整合帳戶。 
   若要將邏輯應用程式連結至具有 Liquid 範本或對應的企業整合帳戶，請遵循下列步驟：

   1. 在企業邏輯應用程式功能表上，選取 [工作流程設定]。

   2. 從 [選取企業整合帳戶] 清單，選取您的企業整合帳戶，並選擇 [儲存]。

     ![將邏輯應用程式連結至企業整合帳戶](./media/logic-apps-enterprise-integration-liquid-transform/link-integration-account.png)

## <a name="test-your-logic-app"></a>測試應用程式邏輯

從 [Postman](https://www.getpostman.com/postman) 或類似的工具將 JSON 輸入張貼至邏輯應用程式。 從邏輯應用程式轉換的 JSON 輸出如下列範例所示：
  
![範例輸出](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontojson.png)

## <a name="more-liquid-action-examples"></a>更多 Liquid 動作範例
Liquid 並非限用於 JSON 轉換。 以下其他可使用 Liquid 的轉換動作。

* JSON 轉換為文字
  
  以下是此範例所使用的 Liquid 範本：
   
   ``` json
   {{content.firstName | Append: ' ' | Append: content.lastName}}
   ```
   以下是範例輸入和輸出：
  
   ![JSON 轉換為文字的範例輸出](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontotext.png)

* XML 轉換為 JSON
  
  以下是此範例所使用的 Liquid 範本：
   
   ``` json
   [{% JSONArrayFor item in content -%}
        {{item}}
    {% endJSONArrayFor -%}]
   ```
   以下是範例輸入和輸出：

   ![XML 轉換為 JSON 的範例輸出](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltojson.png)

* XML 轉換為文字
  
  以下是此範例所使用的 Liquid 範本：

   ``` json
   {{content.firstName | Append: ' ' | Append: content.lastName}}
   ```

   以下是範例輸入和輸出：

   ![XML 轉換為文字的範例輸出](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltotext.png)

## <a name="next-steps"></a>後續步驟

* [深入了解企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md "了解企業整合套件")  
* [深入了解對應](../logic-apps/logic-apps-enterprise-integration-maps.md "了解企業整合對應")  

