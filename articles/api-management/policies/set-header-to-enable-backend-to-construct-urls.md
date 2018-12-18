---
title: Azure API 管理原則範例 - 新增 Forwarded 標頭 | Microsoft Docs
description: Azure API 管理原則範例 - 示範如何在輸入要求中新增 Forwarded 標頭，以允許後端 API 建構適當的 URL。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/13/2017
ms.author: apimpm
ms.openlocfilehash: a9dfcbf4be4b659d761d66d67d2ae4c7b70a245e
ms.sourcegitcommit: d8ffb4a8cef3c6df8ab049a4540fc5e0fa7476ba
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36284159"
---
# <a name="add-a-forwarded-header"></a>新增 Forwarded 標頭

本文會說明 Azure API 管理原則範例，示範如何在輸入要求中新增 Forwarded 標頭，以允許後端 API 建構適當的 URL。 若要設定或編輯原則程式碼，請依照[設定或編輯原則](../set-edit-policies.md)中所述的步驟執行。 若要查看其他範例，請參閱[原則範例](../policy-samples.md)。

## <a name="code"></a>代碼

將程式碼貼至 [輸入] 區塊。

[!code-xml[Main](../../../api-management-policy-samples/examples/Forward gateway hostname to backend for generating correct urls in responses.policy.xml)]

## <a name="next-steps"></a>後續步驟

深入了解 APIM 原則：

+ [轉換原則](../api-management-transformation-policies.md)
+ [原則範例](../policy-samples.md)
