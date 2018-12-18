---
title: 使用 API 管理中的用戶端憑證驗證保護 API - Azure API 管理 | Microsoft Docs
description: 了解如何使用用戶端憑證來保護對 API 的存取
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/01/2017
ms.author: apimpm
ms.openlocfilehash: 841825923819bdb257e5b5983071d999cca805e9
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2017
ms.locfileid: "26406738"
---
# <a name="how-to-secure-apis-using-client-certificate-authentication-in-api-management"></a>如何使用 API 管理中的用戶端憑證驗證保護 API

API 管理提供以用戶端憑證保護對 API 之存取 (例如，用戶端對 API 管理) 的功能。 目前，您可以檢查用戶端憑證的指紋是否符合想要的值。 您也可以檢查指紋是否符合已上傳到 API 管理的現有憑證。  

如需有關如何使用用戶端憑證保護對 API 後端服務之存取 (例如，API 管理到後端) 的資訊，請參閱[如何使用用戶端憑證驗證來保護後端服務](https://docs.microsoft.com/azure/api-management/api-management-howto-mutual-certificates)

## <a name="checking-the-expiration-date"></a>檢查到期日

您可以設定下面的原則來檢查憑證是否已到期：

```
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.NotAfter < DateTime.Now)" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

## <a name="checking-the-issuer-and-subject"></a>檢查簽發者和主旨

您可以設定下面的原則來檢查用戶端憑證的簽發者和主旨：

```
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName != "expected-subject-name")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

## <a name="checking-the-thumbprint"></a>檢查指紋

您可以設定下面的原則來檢查用戶端憑證的指紋：

```
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "desired-thumbprint")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

## <a name="checking-a-thumbprint-against-certificates-uploaded-to-api-management"></a>檢查指紋是否符合已上傳到 API 管理的憑證

下列範例說明如何檢查用戶端憑證的指紋是否符合已上傳到 API 管理的憑證： 

```
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>

```

## <a name="next-step"></a>後續步驟

*  [如何使用用戶端憑證驗證來保護後端服務](https://docs.microsoft.com/azure/api-management/api-management-howto-mutual-certificates)
*  [如何上傳憑證](https://docs.microsoft.com/azure/api-management/api-management-howto-mutual-certificates#a-namestep1-aupload-a-client-certificate)

