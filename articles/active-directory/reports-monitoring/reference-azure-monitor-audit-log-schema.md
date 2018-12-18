---
title: 解譯 Azure 監視器中的 Azure Active Directory 稽核記錄結構描述 (預覽) | Microsoft Docs
description: 描述在 Azure 監視器中使用的 Azure AD 稽核記錄結構描述 (預覽)
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
ms.openlocfilehash: 3df0087a21547585883f791d4d385025f92580c7
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49395032"
---
# <a name="interpret-the-azure-ad-audit-logs-schema-in-azure-monitor-preview"></a>解譯 Azure 監視器中的 Azure AD 稽核記錄結構描述 (預覽)

此文章說明 Azure 監視器中的 Azure Active Directory (Azure AD) 稽核記錄結構描述。 每個個別記錄項目都會儲存為文字，並格式化為 JSON Blob，如以下兩個範例中所示： 

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-17T00:14:31.2585575Z", 
        "operationName": "Change password (self-service)",
        "operationVersion": "1.0",
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "resultDescription": "None", 
        "durationMs": "-1", 
        "correlationId": "60d5e89a-b890-413f-9e25-a047734afe9f", 
        "identity": "sreens@wingtiptoysonline.com", 
        "Level": "Informational", 
        "location": "WUS", 
        "properties": { 
            "identityType": "UPN", 
            "operationType": "Update", 
            "additionalDetails": "None", 
            "additionalTargets": "", 
            "targetUpdatedProperties": "", 
            "targetResourceType": "UPN__TenantContextID__PUID__ObjectID__ObjectClass", 
            "targetResourceName": "sreens@wingtiptoysonline.com__bf85dc9d-cb43-44a4-80c4-469e8c58249e__1003BFFD9FEB17DB__7a408bdd-7d97-4574-8511-dd747b56465d__User", 
            "auditEventCategory": "UserManagement" 
        } 
    } 
    ] 
} 
```

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-18T19:47:43.0368859Z", 
        "operationName": "Update service principal.", 
        "operationVersion": "1.0", 
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "durationMs": "-1", 
        "callerIpAddress": "<null>", 
        "correlationId": "14916c7a-5a7d-44e8-9b06-74b49efb08ee", 
        "identity": "NA", 
        "Level": "Informational", 
        "properties": { 
            "identityType": "NA", 
            "operationType": "Update", 
            "additionalDetails": {}, 
            "additionalTargets": "", 
            "targetUpdatedProperties": [ 
            { 
                "Name": "Included Updated Properties", 
                "OldValue": null, 
                "NewValue": "" 
            }, 
            { 
                "Name": "TargetId.ServicePrincipalNames", 
                "OldValue": null, 
                "NewValue": "http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29" 
            } 
            ], 
        "targetResourceType": "Other__ObjectID__ObjectClass__Name__AppId__SPN", 
        "targetResourceName": "ServicePrincipal_ea70a262-4da3-440a-b396-9734ddfd9df2__ea70a262-4da3-440a-b396-9734ddfd9df2__ServicePrincipal__Salesforce__cd3ed3de-93ee-400b-8b19-b61ef44a0f29__http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29", 
        "auditEventCategory": "ApplicationManagement" 
      } 
    } 
    ] 
} 
```

## <a name="field-and-property-descriptions"></a>欄位與屬性描述

| 欄位名稱 | 說明 |
|------------|-------------|
| 分析       | 日期和時間 (UTC)。 |
| operationName | 作業的名稱。 |
| operationVersion | 用戶端要求的 REST API 版本。 |
| category | 目前，*Audit* 是唯一支援的值。 |
| tenantId | 和記錄相關聯的租用戶 GUID。 |
| resultType | 作業的結果。 結果可以是 *Success* 或 *Failure*。 |
| resultSignature |  此值未對應，您可以放心地忽略它。 | 
| resultDescription | 結果的其他描述 (若有提供)。 | 
| durationMs |  此值未對應，您可以放心地忽略它。 |
| callerIpAddress | 發出要求之用戶端的 IP 位址。 | 
| correlationId | 用戶端傳遞的選擇性 GUID。 它能協助將用戶端作業和伺服器端作業相互關聯，當您在追蹤跨服務的記錄時它會很有用。 |
| 身分識別 | 當您發出要求時，來自出示之權杖的身分識別。 身分識別可以是使用者帳戶、系統帳戶或服務主體。 |
| 層級 | 訊息類型。 稽核記錄的層級一律是 *Informational*。 |
| location | 資料中心的位置。 |
| properties | 列出與稽核記錄相關的受支援屬性。 如需詳細資訊，請參閱下一個表格。 | 

<br>

| 屬性名稱 | 說明 |
|---------------|-------------|
| AuditEventCategory | 稽核事件的類型。 它可以是 *UserManagement*、*ApplicationManagement* 或其他類型。|
| 識別類型 | 類型可以是 *Application* 或 *User*。 |
| 作業類型 | 類型可以是 *Add*、*Update*、*Delete*。 或 *Other*。 |
| 目標資源類型 | 指定作業執行所在的目標資源類型。 類型可以是 *Application*、*User*、*Role*、*Policy* | 
| 目標資源名稱 | 目標資源的名稱。 它可以是應用程式名稱、角色名稱、使用者主體名稱或服務主體名稱。 |
| additionalTargets | 列出任何特定作業的其他內容。 例如，對於更新作業，舊的值和新的值會列在 *targetUpdatedProperties* 下。 | 

## <a name="next-steps"></a>後續步驟

* [解譯 Azure 監視器中的登入記錄結構描述](reference-azure-monitor-sign-ins-log-schema.md)
* [深入了解 Azure 診斷記錄檔](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)
* [常見問題集和已知問題](concept-activity-logs-azure-monitor.md#frequently-asked-questions)
