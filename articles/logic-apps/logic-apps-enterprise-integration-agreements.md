---
title: B2B 通訊的合約 - Azure Logic Apps | Microsoft Docs
description: 建立使用 Azure Logic Apps 與 Enterprise Integration Pack 的 B2B 交易夥伴通訊合約
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 447ffb8e-3e91-4403-872b-2f496495899d
ms.date: 06/29/2016
ms.openlocfilehash: 09bee10649e2bc0d745e42b8aa13ae9c21df35aa
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43128822"
---
# <a name="partner-agreements-for-b2b-communication-with-azure-logic-apps-and-enterprise-integration-pack"></a>具有 Azure Logic Apps 與企業整合套件的 B2B 的合作夥伴合約

合約可讓企業實體使用業界標準通訊協定順利地進行溝通，而且是企業對企業 (B2B) 通訊的基石。 使用企業整合套件針對邏輯應用程式啟用 B2B 案例時，合約是 B2B 交易合作夥伴之間的通訊協議。 此合約是以合作夥伴想要達成的通訊為基礎，而且是通訊協定或傳輸特定的。

企業整合支援下列三種通訊協定或傳輸標準：

* [AS2](logic-apps-enterprise-integration-as2.md)
* [X12](logic-apps-enterprise-integration-x12.md)
* [EDIFACT](logic-apps-enterprise-integration-edifact.md)

## <a name="why-use-agreements"></a>為什麼要使用合約

使用合約有下列常見優點：

* 可讓不同的組織和企業能夠利用已知格式來交換資訊。
* 進行 B2B 交易時，可以提升效率
* 在建立企業整合應用程式時輕鬆地建立、管理及使用它們

## <a name="how-to-create-agreements"></a>如何建立合約

* [建立 AS2 合約](logic-apps-enterprise-integration-as2.md)
* [建立 X12 合約](logic-apps-enterprise-integration-x12.md)
* [建立 EDIFACT 合約](logic-apps-enterprise-integration-edifact.md)

## <a name="how-to-use-an-agreement"></a>如何使用合約

您可以使用您建立的合約來建立具有 B2B 功能的[邏輯應用程式](logic-apps-overview.md "了解邏輯應用程式")。

## <a name="how-to-edit-an-agreement"></a>如何編輯合約

您可以依照下列步驟來編輯任何合約：

1. 選取包含您要更新之合約的整合帳戶。

2. 選擇 [合約] 圖格。

3. 在 [合約] 刀鋒視窗上，選取合約。

4. 選擇 [編輯]。 進行變更。

5. 若要儲存變更，請選擇 [確定]。

## <a name="how-to-delete-an-agreement"></a>如何刪除合約

您可以依照下列步驟來刪除任何合約：

1. 選取包含您要刪除之合約的整合帳戶。
2. 選擇 [合約] 圖格。
3. 在 [合約] 刀鋒視窗上，選取合約。
4. 選擇 [刪除]。
5. 確認您要刪除選取的合約。

    [合約] 刀鋒視窗不會再顯示已刪除的合約。

## <a name="next-steps"></a>後續步驟
* [建立 AS2 合約](logic-apps-enterprise-integration-as2.md)
