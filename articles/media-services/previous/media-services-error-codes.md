---
title: Azure 媒體服務錯誤代碼 | Microsoft Docs
description: 本主題提供 Azure 媒體服務錯誤代碼的概觀。
author: Juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: d3a62a64-7608-4b17-8667-479b26ba0d6c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/31/2017
ms.author: juliako
ms.openlocfilehash: 8a374393a6a5b3d563a441654b7b5df8a510f304
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783567"
---
# <a name="azure-media-services-error-codes"></a>Azure 媒體服務錯誤代碼
使用 Microsoft Azure 媒體服務時，您可能會根據問題從服務收到 HTTP 錯誤代碼，例如驗證權杖過期到媒體服務中不支援的動作。 以下是媒體服務可能會傳回的「HTTP 錯誤代碼」的清單，以及其可能原因。  

## <a name="400-bad-request"></a>400 不正確的要求
要求包含無效的資訊，並且因為下列其中一個原因遭到拒絕︰

* 指定了不支援的 API 版本。 如需最新版本，請參閱[媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。
* 未指定媒體服務的 API 版本。 如需如何指定 API 版本的詳細資訊，請參閱[媒體服務作業 REST API 參考](https://docs.microsoft.com/rest/api/media/operations/azure-media-services-rest-api-reference) \(英文\)。
  
  > [!NOTE]
  > 如果您使用 .NET 或 Java SDK 來連接到媒體服務，每當您針對媒體服務嘗試和執行某些動作時，會為您指定 API 版本。
  > 
  > 
* 已指定未定義的屬性。 屬性名稱在錯誤訊息中。 只能指定屬於指定實體的屬性。 請參閱 [Azure 媒體服務 REST API 參考](https://docs.microsoft.com/rest/api/media/operations/azure-media-services-rest-api-reference)，以取得實體及其屬性的清單。
* 已指定無效的屬性值。 屬性名稱在錯誤訊息中。 請參閱先前連結以取得有效屬性類型及其值。
* 遺漏必要的屬性值。
* 指定 URL 的一部分包含不正確的值。
* 嘗試更新 WriteOnce 屬性。
* 嘗試使用主要 AssetFile (未指定或無法決定) 建立具有輸入資產的作業。
* 嘗試更新 SAS 定位器。 僅能建立或刪除 SAS 定位器。 可以更新串流定位器。 如需詳細資訊，請參閱[定位器](https://docs.microsoft.com/rest/api/media/operations/locator)。
* 不支援的作業或查詢已提交。

## <a name="401-unauthorized"></a>401 未經授權
由於下列其中一個原因，要求無法驗證 (在可授權之前)︰

* 遺失驗證標頭。
* 不正確的驗證標頭值。
  * 權杖已過期。 
  * 權杖包含無效的簽章。

## <a name="403-forbidden"></a>403 禁止
基於下列原因之一不允許此要求：

* 找不到媒體服務帳戶，或已被刪除。
* 媒體服務帳戶已停用，且要求類型並非 HTTP GET。 服務作業也會傳回 403 回應。
* 驗證權杖不包含使用者的認證資訊︰AccountName 和/或 SubscriptionId。 您可以在 Azure 管理入口網站中，媒體服務帳戶的媒體服務 UI 延伸模組中找到這項資訊。
* 無法存取該資源。
  
  * 嘗試使用無法用於媒體服務帳戶的 MediaProcessor。
  * 嘗試更新媒體服務所定義的 JobTemplate。
  * 嘗試覆寫某些其他媒體服務帳戶的定位器。
  * 嘗試覆寫某些其他媒體服務帳戶的 ContentKey。
* 無法建立資源，因為已達到媒體服務帳戶的服務配額。 如需服務配額的詳細資訊，請參閱[配額和限制](media-services-quotas-and-limitations.md)。

## <a name="404-not-found"></a>404 找不到
基於下列原因之一不允許資源的要求：

* 嘗試更新不存在的實體。
* 嘗試刪除不存在的實體。
* 嘗試建立實體，連結至不存在的實體。
* 嘗試取得不存在的實體。
* 嘗試指定未與媒體服務帳戶相關聯的儲存體帳戶。  

## <a name="409-conflict"></a>409 衝突
基於下列原因之一不允許此要求：

* 多個 AssetFile 在資產內具有指定的名稱。
* 嘗試建立資產中的第二個主要 AssetFile。
* 嘗試建立已使用指定識別碼的 ContentKey。
* 嘗試建立已使用指定識別碼的定位器。
* 多個 IngestManifestFile 具有 IngestManifest 中指定的名稱。
* 嘗試將第二個儲存體加密 ContentKey 連結至儲存體加密資產。
* 嘗試將相同的 ContentKey 連結到資產。
* 嘗試建立資產的定位器，其儲存體容器已遺失或已不再與資產相關聯。
* 嘗試建立資產的定位器，該資產已經有 5 個正在使用中的定位器。 (Azure 儲存體強制一個儲存體容器上有五個共用存取原則的限制。)
* 將資產的儲存體帳戶連結到 IngestManifestAsset，與父項 IngestManifest 所使用的儲存體帳戶不相同。  

## <a name="500-internal-server-error"></a>500 內部伺服器錯誤
在處理要求時，媒體服務遇到錯誤，而導致無法繼續處理。 這可能是下列其中一個原因所造成：

* 無法建立資產或作業，因為媒體服務帳戶的服務配額資訊暫時無法使用。
* 無法建立資產或 IngestManifest blob 儲存體容器，因為帳戶的儲存體帳戶資訊暫時無法使用。
* 其他未預期的錯誤。

## <a name="503-service-unavailable"></a>503 服務無法使用
伺服器目前無法接收要求。 此錯誤可能是由於對服務的大量要求所造成。 媒體服務節流機制會針對向服務發出過多要求的應用程式限制資源使用量。

> [!NOTE]
> 請檢查錯誤訊息和錯誤代碼字串，以取得您得到 503 錯誤原因的詳細資訊。 此錯誤不一定表示節流。
> 
> 

可能的狀態描述為︰

* 「伺服器忙碌中。 先前執行此類型的要求超過 {0} 秒」。
* 「伺服器忙碌中。 每秒可以節流超過 {0} 個要求」。
* 「伺服器忙碌中。 {1} 秒內可以節流超過 {0} 個要求」。

若要處理此錯誤，我們建議使用指數輪詢重試邏輯。 這表示在連續錯誤回應的重試之間使用越來越久的等候。  如需詳細資訊，請參閱[暫時性錯誤處理應用程式區塊](https://msdn.microsoft.com/library/hh680905.aspx)。

> [!NOTE]
> 如果您使用 [Azure Media Services SDK for .Net](https://github.com/Azure/azure-sdk-for-media-services/tree/master)，已由 SDK 實作 503 錯誤的重試邏輯。  
> 
> 

## <a name="see-also"></a>另請參閱
[媒體服務管理錯誤代碼](http://msdn.microsoft.com/library/windowsazure/dn167016.aspx)

## <a name="next-steps"></a>後續步驟
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

