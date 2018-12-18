---
title: 在 Azure API 管理 API 匯入的限制和已知問題 | Microsoft Docs
description: 有關使用 Open API、WSDL 或 WADL 格式匯入 Azure API 管理的已知問題和限制的詳細資料。
services: api-management
documentationcenter: ''
author: vladvino
manager: vlvinogr
editor: ''
ms.assetid: 7a5a63f0-3e72-49d3-a28c-1bb23ab495e2
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/29/2017
ms.author: apipm
ms.openlocfilehash: a9f4a4ed4a8771f32a4d66aed2457a43abb92a63
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39295239"
---
# <a name="api-import-restrictions-and-known-issues"></a>API 匯入的限制和已知問題
## <a name="about-this-list"></a>關於本清單
匯入 API 時，您可能會遇到一些限制或識別問題，必須先解決這些問題，您才能成功匯入。 這篇文章說明這些項目，依 API 的匯入格式整理說明。

## <a name="open-api"> </a>OpenAPI/Swagger
如果您匯入 OpenAPI 文件時收到錯誤，請務必確認您的文件 - 可使用 Azure 入口網站中的設計工具 ([設計] - [前端] - [OpenAPI 規格編輯器])，或使用協力廠商工具 (例如 <a href="http://editor.swagger.io">Swagger 編輯器</a>)。

* 只支援適用於 OpenAPI 的 JSON 格式。
* 整個路徑和查詢所需的參數必須具有唯一的名稱。 (在 OpenAPI 中，參數名稱只需要在某個位置內是唯一的，例如路徑、查詢、標頭。  不過，在 API 管理中，可透過路徑和查詢參數來區分作業 (OpenAPI 並不支援此做法)。 因此，參數名稱必須在整個 URL 範本內是唯一的。)
* 參考的結構描述若使用 **$ref** 屬性，不能再包含其他 **$ref** 屬性。
* **$ref** 指標不可以參考外部檔案。
* **x-ms-paths** 和 **x-servers** 是唯二支援的副檔名。
* 在匯入時系統會忽略自訂副檔名，不會儲存或保留用於匯出。

> [!IMPORTANT]
> 請參閱此[文件](https://blogs.msdn.microsoft.com/apimanagement/2018/04/11/important-changes-to-openapi-import-and-export/)中 OpenAPI 匯入相關的重要資訊和祕訣。

## <a name="wsdl"> </a>WSDL
WSDL 檔案是用來產生 SOAP 傳遞的 API，或作為「SOAP 至 REST」API 的後端。
* **SOAP 繫結** - 僅支援「文件」和「常值」編碼的 SOAP 繫結樣式。 不支援「rpc」樣式或 SOAP 編碼。
* **WSDL:Import** - 不支援此屬性。 客戶應該將匯入的項目合併成一份文件。
* **具有多個部分的訊息** - 不支援這些類型的訊息。
* **WCF wsHttpBinding** - 使用 Windows Communication Foundation 建立的 SOAP 服務應該使用 basicHttpBinding，不支援 wsHttpBinding。
* **MTOM** - 使用 MTOM 的服務「可能」可以運作。 目前不提供官方支援。
* **遞迴** - APIM 不支援遞迴定義的類型 (例如，參考自己的陣列)。

## <a name="wadl"> </a>WADL
目前沒有任何已知的 WADL 匯入問題。
