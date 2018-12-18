---
title: 包含檔案
description: 包含檔案
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 06/29/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 7ed298fc8f13685c4872c4c54ba1e447debea79f
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702625"
---
請確定您已建立服務匯流排命名空間，如[這裡][namespace-how-to]所示。

1. 登入 [Azure 入口網站][azure-portal]。
2. 在入口網站的左方瀏覽窗格中，按一下 [服務匯流排] \(如果您未看見 [服務匯流排]，請按一下 [所有服務])。
3. 按一下要在其中建立佇列的命名空間。 在本例中是 **sbnstest1**。
   
    ![建立佇列][createqueue1]
4. 在命名空間視窗中，按一下 [佇列]，然後在 [佇列] 視窗中，按一下 [+ 佇列]。
   
    ![選取佇列][createqueue2]
5. 輸入佇列 [名稱] 並且讓其他值保留其預設值。
   
    ![選取新增][createqueue3]
6. 在視窗底部按一下 [建立]。

[createqueue1]: ./media/service-bus-create-queue-portal/create-queue1.png
[createqueue2]: ./media/service-bus-create-queue-portal/create-queue2.png
[createqueue3]: ./media/service-bus-create-queue-portal/create-queue3.png

[namespace-how-to]: ../articles/service-bus-messaging/service-bus-create-namespace-portal.md
[azure-portal]: https://portal.azure.com
