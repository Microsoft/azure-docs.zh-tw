---
title: 透過入口網站的 Azure 事件方格訂閱
description: 說明如何透過入口網站建立事件方格訂閱。
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: conceptual
ms.date: 08/17/2018
ms.author: tomfitz
ms.openlocfilehash: 72eaa17e78086a4e5338bb3198ef7471c44b785f
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "40234692"
---
# <a name="subscribe-to-events-through-portal"></a>透過入口網站訂閱事件

本文說明如何透過入口網站建立事件方格訂閱。

## <a name="create-event-subscriptions"></a>建立事件訂閱

若要針對任何一個支援的[事件來源](event-sources.md)建立事件方格訂閱，請使用下列步驟。 本文示範如何建立適用於 Azure 訂用帳戶的事件方格訂閱。

1. 選取 [所有服務]。

   ![選取 [所有服務]](./media/subscribe-through-portal/select-all-services.png)

1. 搜尋**事件方格訂閱**，然後從可用的選項中選取它。

   ![Search](./media/subscribe-through-portal/search.png)

1. 選取 [+ 事件訂用帳戶]。

   ![加入訂閱](./media/subscribe-through-portal/add-subscription.png)

1. 選取您要建立的訂閱類型。 例如，若要訂閱適用於您 Azure 訂用帳戶的事件，請選取 [Azure 訂用帳戶] 和目標訂閱。

   ![選取 Azure 訂用帳戶](./media/subscribe-through-portal/azure-subscription.png)

1. 若要訂閱適用於此事件來源的所有事件類型，請讓 [訂閱所有事件類型] 選項保持勾選狀態。 否則，選取適用於此訂閱的事件類型。

   ![選取事件類型](./media/subscribe-through-portal/select-event-types.png)

1. 提供有關事件訂閱的其他詳細資料，例如，用於處理事件的端點和訂閱名稱。

   ![提供訂閱詳細資料](./media/subscribe-through-portal/provide-subscription-details.png)

## <a name="create-subscription-on-resource"></a>建立資源相關的訂閱

某些事件來源支援透過入口網站介面，針對該資源建立事件訂閱。 選取事件來源，並在左窗格中尋找**事件**。

![提供訂閱詳細資料](./media/subscribe-through-portal/resource-events.png)

入口網站會為您顯示用於建立與該來源相關之事件訂閱的選項。

## <a name="next-steps"></a>後續步驟

* 如需事件傳遞和重試的相關資訊，請參閱[事件格線訊息傳遞與重試](delivery-and-retry.md)。
* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)。
* 若要快速地開始使用 Event Grid，請參閱[使用 Azure Event Grid 建立和路由傳送自訂事件](custom-event-quickstart.md)。
