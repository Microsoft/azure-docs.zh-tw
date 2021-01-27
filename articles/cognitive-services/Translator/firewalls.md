---
title: 在防火牆後方轉譯-Translator
titleSuffix: Azure Cognitive Services
description: Azure 認知服務 Translator 可以使用功能變數名稱或 IP 篩選，在防火牆後方轉譯。
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: lajanuar
ms.openlocfilehash: f5dd72328180574809c812d670f8165ad84963ae
ms.sourcegitcommit: 100390fefd8f1c48173c51b71650c8ca1b26f711
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98897742"
---
# <a name="how-to-translate-behind-ip-firewalls-with-translator"></a>如何使用 Translator 在 IP 防火牆後方轉譯

Translator 可以使用網功能變數名稱稱或 IP 篩選，在防火牆後方轉譯。 網域名稱篩選是慣用的方法。 我們 **不建議** 從已篩選 IP 的防火牆後方執行 Microsoft Translator。 安裝程式未來可能會中斷，恕不另行通知。

## <a name="translator-ip-addresses"></a>Translator IP 位址
從2019年8月21日起，api.cognitive.microsofttranslator.com-翻譯工具的 IP 位址：

* **亞太地區：** 20.40.125.208、20.43.88.240、20.184.58.62、40.90.139.163、104.44.89.44
* **歐洲：** 40.90.138.4、40.90.141.99、51.105.170.64、52.155.218.251
* **北美洲：** 40.90.139.36、40.90.139.2、40.119.2.134、52.224.200.129、52.249.207.163

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [翻譯工具中的 IP 防火牆後方轉譯](reference/v3-0-translate.md)
