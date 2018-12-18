---
title: 包含檔案
description: 包含檔案
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: include
ms.custom: include file
ms.date: 09/27/2018
ms.author: diberry
ms.openlocfilehash: 420757c5b7f6d19bb0abe87232e75497b7b97322
ms.sourcegitcommit: 7bc4a872c170e3416052c87287391bc7adbf84ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48019084"
---
1. 開啟 Visual Studio 2017 Community 版本。
1. 建立新的**主控台應用程式 (.NET Core)** 專案，並且將專案命名為 `QnaMakerQuickstart`。 接受其餘設定的預設值。
1. 在 [方案總管] 中，以滑鼠右鍵按一下專案名稱，選取 [QnaMakerQuickstart]，然後選取 [管理 NuGet 套件]。
1. 在 [NuGet] 視窗中，選取 [瀏覽器]，然後搜尋 **Newtonsoft.JSON** 並安裝套件。 此套件是用來剖析 QnaMaker HTTP 回應傳回的 JSON。 