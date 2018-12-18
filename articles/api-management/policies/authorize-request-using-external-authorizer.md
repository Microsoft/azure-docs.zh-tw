---
title: Azure API 管理原則範例 - 使用外部授權者授權要求 | Microsoft Docs
description: Azure API 管理原則範例 - 示範如何使用外部授權者封裝自訂或傳統驗證/授權邏輯來授權要求。
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
ms.date: 06/06/2018
ms.author: apimpm
ms.openlocfilehash: 70f43a058cfd3818dae1ccfa4b222a7d0d740aee
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979398"
---
# <a name="authorize-requests-using-external-authorizer"></a>使用外部授權者授權要求

本文將說明 Azure API 管理原則範例，示範如何使用外部授權者封裝自訂驗證/授權邏輯來保護 API 存取。 若要設定或編輯原則程式碼，請依照[設定或編輯原則](../set-edit-policies.md)中所述的步驟執行。 若要查看其他範例，請參閱[原則範例](../policy-samples.md)。

## <a name="policy"></a>原則

將程式碼貼至 [輸入] 區塊。

[!code-xml[Main](../../../api-management-policy-samples/examples/Authorize requests using external authorizer.policy.xml)]

## <a name="next-steps"></a>後續步驟

深入了解 APIM 原則：

+ [存取限制原則](../api-management-access-restriction-policies.md)
+ [原則範例](../policy-samples.md)
