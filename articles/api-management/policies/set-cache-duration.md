---
title: Azure API 管理原則範例 - 設定回應快取持續時間 | Microsoft Docs
description: Azure API 管理原則範例 - 示範如何使用後端所傳送之 Cache-Control 標頭中的 maxAge 值來設定回應快取持續時間。
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
ms.openlocfilehash: 8f7126f5cd6bf6f142c603e4b1baee4a6c20dea2
ms.sourcegitcommit: d8ffb4a8cef3c6df8ab049a4540fc5e0fa7476ba
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36287546"
---
# <a name="set-response-cache-duration"></a>設定回應快取持續時間

本文會說明 Azure API 管理原則範例，示範如何使用後端所傳送之 Cache-Control 標頭中的 maxAge 值來設定回應快取持續時間。 若要設定或編輯原則程式碼，請依照[設定或編輯原則](../set-edit-policies.md)中所述的步驟執行。 若要查看其他範例，請參閱[原則範例](../policy-samples.md)。

## <a name="policy"></a>原則

將程式碼貼至 [輸入] 區塊。

[!code-xml[Main](../../../api-management-policy-samples/examples/Set cache duration using response cache control header.policy.xml)]

## <a name="next-steps"></a>後續步驟

深入了解 APIM 原則：

+ [轉換原則](../api-management-transformation-policies.md)
+ [原則範例](../policy-samples.md)

