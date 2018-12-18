---
title: 新增 B2B 整合帳戶的交易夥伴 - Azure Logic Apps | Microsoft Docs
description: 在採用 Enterprise Integration Pack 的 Logic Apps 中為您的整合帳戶建立交易夥伴
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: b179325c-a511-4c1b-9796-f7484b4f6873
ms.date: 07/08/2016
ms.openlocfilehash: 20ca5e06cd1cd0d0abfe6d31f622cd6b61b4178f
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125256"
---
# <a name="add-trading-partners-for-integration-accounts-in-azure-logic-apps-with-enterprise-integration-pack"></a>在採用 Enterprise Integration Pack 的 Azure Logic Apps 中為整合帳戶新增交易夥伴

合作夥伴是參與企業對企業 (B2B) 交易及在彼此之間交換訊息的實體。 在您可以建立這些交易中代表您與其他組織的合作夥伴之前，你們雙方必須先共用可識別及驗證彼此所傳送訊息的資訊。 在您討論這些詳細資料並準備開始您的商業關係之後，您可以在您的整合帳戶中建立代表你們雙方的合作夥伴。

## <a name="what-roles-do-partners-play-in-your-integration-account"></a>合作夥伴在您的整合帳戶中扮演什麼角色？

為定義在合作夥伴之間交換之訊息的相關詳細資料，您必須在那些合作夥伴之間建立合約。 然而，在您可以建立合約之前，至少需要在整合帳戶中新增兩個以上的合作夥伴。 您的組織必須是與**主機合作夥伴**相關之合約的一部分。 另一個合作夥伴 (或稱為**來賓合作夥伴**) 代表與您的組織交換訊息的組織。 來賓合作夥伴可以是另一家公司，或甚至是您自己組織中的部門。

在您新增這些合作夥伴之後，您可以建立合約。

接收和傳送設定是主控夥伴觀點導向的。 例如，合約中的接收設定會決定主控夥伴如何接收來賓夥伴所傳送的訊息。 同樣地，合約中的傳送設定會指出主控夥伴如何將訊息傳送給來賓夥伴。

## <a name="create-partner"></a>建立合作夥伴

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 在主要 Azure 功能表上，選取 [所有服務]。 在搜尋方塊中輸入「整合」，然後選取 [整合帳戶]。

   ![尋找整合帳戶](./media/logic-apps-enterprise-integration-partners/account-1.png)

3. 在 [整合帳戶] 底下，選取要將合作夥伴新增到其中的整合帳戶。

   ![選取企業整合帳戶](./media/logic-apps-enterprise-integration-partners/account-2.png)

4. 選擇 [合作夥伴] 圖格。

   ![選擇「合作夥伴」](./media/logic-apps-enterprise-integration-partners/partner-1.png)

5. 在 [合作夥伴] 底下，選擇 [新增]。

   ![選擇 [新增]](./media/logic-apps-enterprise-integration-partners/partner-2.png)

6. 輸入合作夥伴的名稱，然後選取 [辨識符號]。 輸入用來識別應用程式所接收之文件的**值**。 完成時，選擇 [確定]。

   ![新增合作夥伴詳細資料](./media/logic-apps-enterprise-integration-partners/partner-3.png)

7. 再次選擇 [合作夥伴] 圖格。

   ![選擇「合作夥伴」圖格](./media/logic-apps-enterprise-integration-partners/partner-5.png)

   您的新合作夥伴隨即顯示。 

   ![檢視新的合作夥伴](./media/logic-apps-enterprise-integration-partners/partner-6.png)

## <a name="edit-partner"></a>編輯合作夥伴

1. 在 [Azure 入口網站](https://portal.azure.com)中，尋找並選取整合帳戶。 選擇 [合作夥伴] 圖格。

   ![選擇「合作夥伴」圖格](./media/logic-apps-enterprise-integration-partners/edit.png)

2. 在 [合作夥伴] 底下，選取您要編輯的合作夥伴。

   ![選取要刪除的合作夥伴](./media/logic-apps-enterprise-integration-partners/edit-1.png)

3. 在 [更新合作夥伴] 底下，進行您所需的變更。
完成之後，選擇 [儲存]。 

   ![進行變更並儲存](./media/logic-apps-enterprise-integration-partners/edit-2.png)

   若要取消您的變更，請選取 [捨棄]。

## <a name="delete-partner"></a>刪除合作夥伴

1. 在 [Azure 入口網站](https://portal.azure.com)中，尋找並選取整合帳戶。 選擇 [合作夥伴] 圖格。

   ![選擇「合作夥伴」圖格](./media/logic-apps-enterprise-integration-partners/delete.png)

2. 在 [合作夥伴] 底下，選取您要刪除的合作夥伴。
選擇 [刪除]。

   ![刪除合作夥伴](./media/logic-apps-enterprise-integration-partners/delete-1.png)

## <a name="next-steps"></a>後續步驟

* [深入了解合約](../logic-apps/logic-apps-enterprise-integration-agreements.md "了解企業整合合約")  

