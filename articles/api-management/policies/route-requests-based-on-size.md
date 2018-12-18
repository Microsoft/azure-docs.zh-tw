---
title: Azure API 管理原則範例 - 根據要求主體的大小進行要求的路由傳送 | Microsoft Docs
description: Azure API 管理原則範例 - 示範如何根據要求主體的大小進行要求的路由傳送。
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
ms.openlocfilehash: a93e1d9fecea59ebb68c512b96c8381b5b1a9346
ms.sourcegitcommit: d8ffb4a8cef3c6df8ab049a4540fc5e0fa7476ba
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36284755"
---
# <a name="route-the-request-based-on-the-size-of-its-body"></a>根據要求主體的大小進行要求的路由傳送

本文會說明 Azure API 管理原則範例，示範如何根據要求主體的大小進行要求的路由傳送。 若要設定或編輯原則程式碼，請依照[設定或編輯原則](../set-edit-policies.md)中所述的步驟執行。 若要查看其他範例，請參閱[原則範例](../policy-samples.md)。

## <a name="policy"></a>原則

將程式碼貼至 [輸入] 區塊。

[!code-xml[Main](../../../api-management-policy-samples/examples/Route requests based on size.policy.xml)]

## <a name="next-steps"></a>後續步驟

深入了解 APIM 原則：

+ [轉換原則](../api-management-transformation-policies.md)
+ [原則範例](../policy-samples.md)

