---
title: Azure API 管理原則範例 | Microsoft Docs
description: 了解可在「Azure API 管理」中使用的原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: cflower
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: sample
ms.date: 10/31/2017
ms.author: apimpm
ms.custom: mvc
ms.openlocfilehash: c58396379db2d9073c1f7d3a4c3aa68dfd06079e
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34830178"
---
# <a name="api-management-policy-samples"></a>API 管理原則範例

[原則](api-management-howto-policies.md)是系統的強大功能，可讓發行者透過設定來變更 API 的行為。 原則是陳述式的集合，會因 API 的要求或回應循序執行。 下表包含範例的連結，並提供每個範例的簡短描述。

|                                                                                                                                                                      |                                                                                                                                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **輸入原則**                                                                                                                                                 |                                                                                                                                                                                                                             |
| [新增 Forwarded 標頭以允許後端 API 建構適當的 URL](./policies/set-header-to-enable-backend-to-construct-urls.md?toc=api-management/toc.json) | 示範如何在輸入要求中新增 Forwarded 標頭，以允許後端 API 建構適當的 URL。                                                                                                        |
| [新增包含相互關聯識別碼的標頭](./policies/add-correlation-id.md?toc=api-management/toc.json)                                                             | 示範如何將包含相互關聯識別碼的標頭新增至輸入要求。                                                                                                                                        |
| [將功能新增至後端服務並快取回應](./policies/cache-response.md?toc=api-management/toc.json)                                             | 示範如何將功能新增至後端服務。 例如，在天氣預報 API 中接受地點的名稱，而不是緯度和經度。                                                                    |
| [根據 JWT 宣告授權存取](./policies/authorize-request-based-on-jwt-claims.md?toc=api-management/toc.json)                                              | 示範如何在以 JWT 宣告為基礎的 API 上授權存取特定的 HTTP 方法。                                                                                                                                       |
| [使用外部授權者授權要求](./policies/authorize-request-using-external-authorizer.md)                                                   | 示範如何使用外部授權者來保護 API 存取。                                                                                                                                                               |
| [使用 Google OAuth 權杖授權存取](./policies/use-google-as-oauth-token-provider.md?toc=api-management/toc.json)                                            | 示範如何使用 Google 作為 OAuth 權杖提供者來授權存取您的端點。                                                                                                                                    |
| [產生共用存取簽章並將要求轉送到 Azure 儲存體](./policies/generate-shared-access-signature.md?toc=api-management/toc.json)                  | 示範如何使用運算式來產生[共用存取簽章](https://docs.microsoft.com/azure/storage/storage-dotnet-shared-access-signature-part-1)，並使用 rewrite-uri 原則來將要求轉送到 Azure 儲存體。 |
| [從 AAD 取得 OAuth2 存取權杖並將它轉送到後端](./policies/use-oauth2-for-authorization.md?toc=api-management/toc.json)                             | 提供使用 OAuth2 在閘道與後端之間進行授權的範例。 此範例示範如何從 AAD 取得存取權杖並將它轉送到後端。                                                    |
| [使用傳送要求原則從 SAP 閘道取得 X-CSRF 權杖](./policies/get-x-csrf-token-from-sap-gateway.md?toc=api-management/toc.json)                           | 示範如何實作許多 API 所使用的 X-CSRF 模式。 這是 SAP 閘道特有的範例。                                                                                                                           |
| [根據要求主體的大小進行要求的路由傳送](./policies/route-requests-based-on-size.md?toc=api-management/toc.json)                                            | 示範如何根據要求主體的大小進行要求的路由傳送。                                                                                                                                                       |
| [將要求內容資訊傳送到後端服務](./policies/send-request-context-info-to-backend-service.md?toc=api-management/toc.json)                    | 示範如何將一些內容資訊傳送到後端服務以進行記錄或處理。                                                                                                                                |
| [設定回應快取持續時間](./policies/set-cache-duration.md?toc=api-management/toc.json)                                                                          | 示範如何使用後端所傳送之 Cache-Control 標頭中的 maxAge 值來設定回應快取持續時間。                                                                                                             |
| **輸出原則**                                                                                                                                                |                                                                                                                                                                                                                             |
| [篩選回應內容](./policies/filter-response-content.md?toc=api-management/toc.json)                                                                         | 示範如何根據與要求相關聯的產品，從回應承載篩選資料元素。                                                                                                        |
| **錯誤時的原則**                                                                                                                                                |                                                                                                                                                                                                                             |
| [將錯誤記錄到 Stackify](./policies/log-errors-to-stackify.md?toc=api-management/toc.json)                                                                           | 示範如何新增錯誤記錄原則，以將錯誤傳送到 Stackify 進行記錄。                                                                                                                                            |
