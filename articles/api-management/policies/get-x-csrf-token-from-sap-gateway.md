---
title: Azure API 管理原則範例 - 實作 X-CSRF 模式 | Microsoft Docs
description: Azure API 管理原則範例 - 示範如何實作許多 API 所使用的 X-CSRF 模式。 這是 SAP 閘道特有的範例。
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
ms.openlocfilehash: 3a2067836a1488d117dced96f3935f2d1f8b1b48
ms.sourcegitcommit: e34afd967d66aea62e34d912a040c4622a737acb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2018
ms.locfileid: "36946018"
---
# <a name="implement-x-csrf-pattern"></a>實作 X-CSRF 模式

本文會說明 Azure API 管理原則範例，示範如何實作許多 API 所使用的 X-CSRF 模式。 這是 SAP 閘道特有的範例。 若要設定或編輯原則程式碼，請依照[設定或編輯原則](../set-edit-policies.md)中所述的步驟執行。 若要查看其他範例，請參閱[原則範例](../policy-samples.md)。

## <a name="policy"></a>原則

將程式碼貼至 [輸入] 區塊。

[!code-xml[Main](../../../api-management-policy-samples/examples/Get X-CSRF token from SAP gateway using send request.policy.xml)]

## <a name="next-steps"></a>後續步驟

深入了解 APIM 原則：

+ [轉換原則](../api-management-transformation-policies.md)
+ [原則範例](../policy-samples.md)

