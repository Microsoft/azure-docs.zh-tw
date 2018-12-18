---
title: 建立 B2B 企業整合 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Logic Apps 與 Enterprise Integration Pack 接收 B2B 資料
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 20fc3722-6f8b-402f-b391-b84e9df6fcff
ms.date: 07/08/2016
ms.openlocfilehash: ad7a29f4a554d599b17576921542b1ac6e403911
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43127759"
---
# <a name="receive-b2b-data-with-azure-logic-apps-and-enterprise-integration-pack"></a>使用 Azure Logic Apps 與 Enterprise Integration Pack 接收 B2B 資料

建立具有合作夥伴與合約的整合帳戶之後，您就可以使用[企業整合套件](logic-apps-enterprise-integration-overview.md)為您的邏輯應用程式建立企業對企業 (B2B) 工作流程。

## <a name="prerequisites"></a>必要條件

若要使用 AS2 和 X12 動作，您必須有企業整合帳戶。 了解[如何建立企業整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md)。

## <a name="create-a-logic-app-with-b2b-connectors"></a>使用 B2B 連接器建立邏輯應用程式

依照這些步驟建立使用 AS2 與 X12 動作從交易合作夥伴接收資料的 B2B 邏輯應用程式：

1. 建立邏輯應用程式，然後[將應用程式連結到您的整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md)。

2. 將 [要求 - 收到 HTTP 要求時]  觸發程序新增到您的邏輯應用程式。

    ![](./media/logic-apps-enterprise-integration-b2b/flatfile-1.png)

3. 若要新增 [將 AS2 解碼] 動作，請選取 [新增動作]。

    ![](./media/logic-apps-enterprise-integration-b2b/transform-2.png)

4. 若要篩選所有動作以尋找您要的動作，請在搜尋方塊中輸入 **as2**。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-5.png)

5. 選取 [AS2 - 將 AS2 訊息解碼] 動作。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-6.png)

6. 新增要做為輸入使用的 [主體]。 在此範例中，選取觸發邏輯應用程式的 HTTP 要求主體。 或者，輸入會在 [標頭] 欄位輸入標頭的運算式：

    @triggerOutputs()['headers']

7. 針對 AS2 新增要求的 [標頭]，您可以在 HTTP 要求標頭中找到標頭。 在此範例中，選取觸發邏輯應用程式的 HTTP 要求標頭。

8. 現在，新增「將 X12 訊息解碼」動作。 選取 [新增動作]。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-9.png)

9. 若要篩選所有動作以尋找您要的動作，請在搜尋方塊中輸入 **x12**。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-10.png)

10. 選取 [X12 - 將 X12 訊息解碼] 動作。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-as2message.png)

11. 現在您必須指定此動作的輸入。 此輸入是上一個 AS2 動作的輸出。

    實際訊息內容是 JSON 物件格式且使用 base64 編碼，因此您必須將運算式指定為輸入。 
    在 [要解碼的 X12 一般檔案訊息] 輸入欄位中輸入下列運算式：
    
    @base64ToString(body('Decode_AS2_message')?['AS2Message']?['Content'])

    現在，請新增步驟以將接收自交易夥伴的 X12 資料解碼，並輸出 JSON 物件中的一些項目。 
    若要通知合作夥伴已收到資料，您可以在 HTTP 回應動作中送回包含 AS2 郵件處置通知 (MDN) 的回應。

12. 若要新增 [回應] 動作，請選擇 [新增動作]。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-14.png)

13. 若要篩選所有動作以尋找您要的動作，請在搜尋方塊中輸入**回應**。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-15.png)

14. 選取 [回應] 動作。

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-16.png)

15. 若要從 [將 X12 訊息解碼] 動作的輸出存取 MDN，請使用此運算式設定回應**主體**：

    @base64ToString(body('Decode_AS2_message')?['OutgoingMdn']?['Content'])

    ![](./media/logic-apps-enterprise-integration-b2b/b2b-17.png)  

16. 儲存您的工作。

    ![](./media/logic-apps-enterprise-integration-b2b/transform-5.png)  

您現在已完成 B2B 邏輯應用程式設定。 在真實世界應用程式中，您可能想要在企業營運 (LOB) 應用程式或資料存放區中儲存已解碼的 X12 資料。 若要連線到您自己的 LOB 應用程式並在您的邏輯應用程式中使用這些 API，您可以新增進一步的動作或撰寫自訂 API。

## <a name="features-and-use-cases"></a>功能和使用案例

* AS2 與 X12 解碼與編碼動作可讓您在邏輯應用程式中使用業界標準通訊協定在交易合作夥伴之間交換資料。
* 若要與交易合作夥伴交換資料，您可以使用 AS2 或 X12 其中一種或將兩種搭配使用。
* B2B 動作可協助您在整合帳戶中輕鬆地建立合作夥伴與合約，並在邏輯應用程式中取用。
* 當您使用其他動作延伸您邏輯應用程式的功能時，您可以在其他應用程式與服務 (例如 SalesForce) 之間傳送及接收資料。

## <a name="learn-more"></a>深入了解
[深入了解企業整合套件](logic-apps-enterprise-integration-overview.md)
