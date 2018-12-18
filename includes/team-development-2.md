---
title: 包含檔案
description: 包含檔案
ms.custom: include file
services: azure-dev-spaces
ms.service: azure-dev-spaces
ms.component: azds-kubernetes
author: ghogen
ms.author: ghogen
ms.date: 05/11/2018
ms.topic: include
manager: douge
ms.openlocfilehash: 8580e19f1955becb2cfaee13742054af1a9edb7f
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47410439"
---
### <a name="run-the-service"></a>執行服務

1. 按 F5 (或在終端機視窗中輸入 `azds up`) 來執行服務。 服務會自動在您新選取的空間 `default/scott` 中執行。 
1. 您可以藉由再次執行 `azds list-up`，確認您的服務是在其本身的空間中執行。 首先，您會注意到 `mywebapi` 的執行個體現在是在 `default/scott` 空間中執行 (在 `default` 中執行的版本仍在執行中，但是不會列出)。 如果您執行 `azds list-uris`，您會注意到 `webfrontend` 的存取點 URL 前面加上文字 "scott.s."。 此 URL 是 `default/scott` 空間專屬的。 此特殊 URL 表示傳送到  "scott URL" 的要求會嘗試先路由傳送到 `default/scott` 空間中的服務，但若失敗，則會切換回 `default` 空間中的服務。

```
Name                      DevSpace  Type     Updated  Status
------------------------  --------  -------  -------  -------
mywebapi                  scott     Service  3m ago   Running
mywebapi-bb4f4ddd8-sbfcs  scott     Pod      3m ago   Running
webfrontend               default   Service  26m ago  Running
```

```
Uri                                                            Status
-------------------------------------------------------------  ---------
http://localhost:53831 => mywebapi.scott:80                    Tunneled
http://webfrontend.6364744826e042319629.canadaeast.aksapp.io/  Available
```

![](../articles/dev-spaces/media/common/space-routing.png)

Azure Dev Spaces 的這項內建功能可讓您在共用空間中測試程式碼，而不需要每位開發人員在其空間中重新建立服務的完整堆疊。 這個路由需要您的應用程式程式碼才能轉送傳播標頭，如本指南的上一個步驟所述。

### <a name="test-code-in-a-space"></a>在空間中測試程式碼
若要測試新版的 `mywebapi` 與 `webfrontend` 的搭配運作情形，請開啟瀏覽器並進入 `webfrontend` 的公用存取點 URL，然後移至 [關於] 頁面。 您應該會看到新訊息顯示。

現在，移除 URL 的 "scott.s." 部分，然後重新整理瀏覽器。 您應該會看到舊的行為 (使用在 `default` 中執行的 `mywebapi` 版本)。
