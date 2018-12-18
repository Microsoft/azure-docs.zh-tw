---
title: 解譯 Azure 監視器中的 Azure Active Directory 登入記錄結構描述 (預覽) | Microsoft Docs
description: 描述在 Azure 監視器中使用的 Azure AD 登入記錄結構描述 (預覽)
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 07/13/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: b9ad4e69c0693bc856789c52870a588671573b5c
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42144241"
---
# <a name="interpret-the-azure-ad-sign-in-logs-schema-in-azure-monitor-preview"></a>解譯 Azure 監視器中的 Azure AD 登入記錄結構描述 (預覽)

此文章說明 Azure 監視器中的 Azure Active Directory (Azure AD) 登入記錄結構描述。 大部分與登入相關的資訊都會在 `records` 物件的 *Properties* 屬性下提供。

```json
{ 
    "records": [ 
    { 
        "time": "2018-05-16T16:09:58.4634578Z", 
        "resourceId": null, 
        "operationName": "Sign-in activity", 
        "operationVersion": "1.0", 
        "category": "SignIn", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "50140", 
        "resultSignature": "None", 
        "resultDescription": "Other", 
        "durationMs": 0, 
        "callerIpAddress": "167.220.0.158", 
        "correlationId": "13e19598-e040-487f-bd32-d38a2cd75d9a", 
        "identity": "arvind harinder", 
        "Level": 4, 
        "location": "US", 
        "properties": { 
            "id": "0782c515-08b6-4029-a65c-29d9a3d20800", 
            "createdDateTime": "2018-05-16T16:09:58.4634578+00:00", 
            "userDisplayName": "Arvind Harinder", 
            "userPrincipalName": "ah@wingtiptoysonline.onmicrosoft.com", 
            "userId": "5b9f356d-9592-42fd-9ec4-d70963909534", 
            "appId": "c44b4083-3bb0-49c1-b47d-974e53cbdf3c", 
            "appDisplayName": "Azure Portal", 
            "ipAddress": "167.220.0.158", 
            "status": { 
                "errorCode": 50140, 
                "failureReason": "Other" 
            }, 
            "clientAppUsed": "Browser", 
            "deviceDetail": { 
                "operatingSystem": "Windows 10", 
                "browser": "Chrome 66.0.3359" 
            }, 
            "location": { 
                "city": "Sammamish", 
                "state": "Washington", 
                "countryOrRegion": "US", 
                "geoCoordinates": { 
                    "latitude": 47.66630935668945, 
                    "longitude": -122.09821319580078 
                } 
            }, 
            "correlationId": "13e19598-e040-487f-bd32-d38a2cd75d9a", 
            "conditionalAccessStatus": 2, 
            "conditionalAccessPolicies": [ 
            { 
                "id": "de7e60eb-ed89-4d73-8205-2227def6b7c9", 
                "displayName": "[billg] SharePoint limited access policy", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "7412a2d8-cbb1-4f1c-96cf-8410b4b8b37b", 
                "displayName": "[BillG] AIP MFA Policy", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "727ed8ea-059d-4d8f-aba5-c1dc500e8b06", 
                "displayName": "[billg] mfa for mail", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "6701123a-b4c6-48af-8565-565c8bf7cabc", 
                "displayName": "Medium signin risk block", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "fbafa2da-cf7f-4ec3-83cf-281188e53f76", 
                "displayName": "Require MFA for admins [Ignite talk] ", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "15339054-709d-4e06-a9ec-342bf043ea56", 
                "displayName": "Enhanced proofing for Azure portal [Ignite talk]", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "2ff9436f-bc72-4ce6-b17e-e7e51153146e", 
                "displayName": "[calebb] AIP policy", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "46ab586b-9447-4847-a889-e60705d96e56", 
                "displayName": "Test policy, OR", 
                "enforcedGrantControls": [], 
                "enforcedSessionControls": [], 
                "result": 3 
            }, 
            { 
                "id": "ceb6e17e-a5d0-4b3a-a150-6c2be2d5b0e9", 
                "displayName": "mm policy with Duo", 
                "enforcedGrantControls": [ 
                    "Require Duo Mfa" 
                ], 
            "enforcedSessionControls": [], 
            "result": 2 
            }, 
            ], 
            "isRisky": false 
            } 
        } 
    } 
```

## <a name="field-descriptions"></a>欄位描述

| 欄位名稱 | 說明 |
|------------|-------------|
| 時間 | 日期和時間 (UTC)。 |
| ResourceId | 此值未對應，您可以放心地略過此欄位。  |
| OperationName | 針對登入，這個值一律是 *Sign-in activity*。 |
| OperationVersion | 用戶端要求的 REST API 版本。 |
| 類別 | 針對登入，這個值一律是 *SignIn*。 | 
| TenantId | 和記錄相關聯的租用戶 GUID。 |
| ResultType | 登入作業的結果可以是 *Success* 或 *Failure*。 | 
| ResultSignature | 包含登入作業的錯誤碼 (如果有)。 |
| ResultDescription | 提供登入作業的錯誤描述。 |
| DurationMs |  此值未對應，您可以放心地略過此欄位。|
| CallerIpAddress | 發出要求之用戶端的 IP 位址。 | 
| CorrelationId | 用戶端傳遞的選擇性 GUID。 此值能協助將用戶端作業和伺服器端作業相互關聯，當您在追蹤跨服務的記錄時它會很有用。 |
| 身分識別 | 當您發出要求時，來自出示之權杖的身分識別。 它可以是使用者帳戶、系統帳戶或服務主體。 |
| Level | 提供訊息的類型。 針對稽核，它一律是 *Informational*。 |
| 位置 | 提供登入活動的位置。 |
| properties | 列出與登入相關的所有屬性。如需詳細資訊，請參閱 [Microsoft 圖形 API 參考](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/signin) \(英文\)。 此結構描述使用與登入資源相同的屬性名稱，以提高可讀性。

## <a name="next-steps"></a>後續步驟

* [解譯 Azure 監視器中的稽核記錄結構描述](reference-azure-monitor-audit-log-schema.md)
* [深入了解 Azure 診斷記錄檔](../../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)
