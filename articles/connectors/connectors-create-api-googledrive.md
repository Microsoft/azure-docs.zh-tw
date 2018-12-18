---
title: 連線到 Google Drive - Azure Logic Apps | Microsoft Docs
description: 使用 Google Drive REST API 和 Azure Logic Apps 建立及管理檔案
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 11/07/2016
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 4994687afbdd23f0265138cd1d4eff53bb47c163
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35295878"
---
# <a name="get-started-with-the-google-drive-connector"></a>開始使用 Google 雲端硬碟連接器
連線到 Google 雲端硬碟來建立檔案、取得資料列等等。 您可以利用 Google 雲端硬碟來： 

* 根據您透過搜尋所取得的資料，來建置您的商務流程。 
* 使用動作來搜尋圖像、搜尋新聞等等。 這些動作會收到回應，然後輸出能讓其他動作使用的資料。 舉例來說，您可以搜尋某支影片，然後利用 Twitter 把該影片張貼在某個 Twitter 摘要上。

您可以從建立邏輯應用程式立即開始，請參閱[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="create-the-connection-to-google-drive"></a>建立至 Google 雲端硬碟的連線
當您將這個 API 新增到邏輯應用程式時，您必須授權邏輯應用程式，使其能夠連線到您的 Google 雲端硬碟。

> [!INCLUDE [Steps to create a connection to googledrive](../../includes/connectors-create-api-googledrive.md)]
> 
> 

當您建立連線之後，請輸入 Google 雲端硬碟的屬性，例如資料夾路徑或檔案名稱。 

## <a name="connector-specific-details"></a>連接器特定的詳細資料

檢視 Swagger 中定義的任何觸發程序和動作，另請參閱[連接器詳細資料](/connectors/googledrive/)的所有限制。

## <a name="more-connectors"></a>其他連接器
返回 [API 清單](apis-list.md)。
