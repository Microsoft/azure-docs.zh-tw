---
title: 連線到 Office 365 使用者 - Azure Logic Apps | Microsoft Docs
description: 使用 Office 365 使用者 REST API 和 Azure Logic Apps 管理使用者設定檔
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 08/18/2016
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: f030ac07dc1615c435c1a110836d7a03ab8a8546
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35295603"
---
# <a name="get-started-with-the-office-365-users-connector"></a>開始使用 Office 365 使用者連接器
連接至 Office 365 使用者，以取得設定檔、搜尋使用者等等。 您可以利用 Office 365 使用者來：

* 根據您從 Office 365 使用者所取得的資料，來建置您的商務流程。 
* 使用可取得直屬員工、取得管理員的使用者設定檔等等的動作。 這些動作會收到回應，然後輸出能讓其他動作使用的資料。 例如，取得某人的直屬員工，然後利用此資訊更新 SQL Azure 資料庫。 

您可以從建立邏輯應用程式立即開始，請參閱[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="create-a-connection-to-office-365-users"></a>建立至 Office 365 使用者的連線
當您將這個連接器新增到邏輯應用程式時，您必須登入您的 Office 365 使用者帳戶，並允許邏輯應用程式連線到您的帳戶。

> [!INCLUDE [Steps to create a connection to Office 365 Users](../../includes/connectors-create-api-office365users.md)]
> 
> 

連線建立之後，您需要輸入 Office 365 使用者屬性，像是使用者識別碼。 本文的 **REST API 參考**一節說明這些屬性。

## <a name="connector-specific-details"></a>連接器特定的詳細資料

檢視 Swagger 中定義的任何觸發程序和動作，另請參閱[連接器詳細資料](/connectors/officeusers/)的所有限制。

## <a name="more-connectors"></a>其他連接器
返回 [API 清單](apis-list.md)。