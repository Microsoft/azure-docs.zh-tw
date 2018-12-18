---
title: Azure 媒體服務 v3 常見問題集 | Microsoft Docs
description: 本文提供 Azure 媒體服務 v3 常見問題集的解答。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/29/2018
ms.author: juliako
ms.openlocfilehash: 098a34aba8e5ce23f64d4bb07e3b9622aa2adb8e
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37110407"
---
# <a name="azure-media-services-v3-preview-frequently-asked-questions"></a>Azure 媒體服務 v3 (預覽) 常見問題集

本文提供 Azure 媒體服務 (AMS) v3 常見問題集的解答。

## <a name="can-i-use-the-azure-portal-to-manage-v3-resources"></a>是否可以使用 Azure 入口網站來管理 v3 資源？

尚未提供。 您可以使用其中一個受支援的 SDK。 請參閱此文件集內的教學課程和範例。

## <a name="is-there-an-api-for-configuring-media-reserved-units"></a>是否有用於設定媒體保留單元的 API？

媒體服務小組將會在 v3 剔除 RU。 但所需的服務工作尚未完成。 在那之前，客戶必須使用 Azure 入口網站或 AMS v2 API 來設定 RU (如[調整媒體處理](../previous/media-services-scale-media-processing-overview.md)所述)。 

使用 **VideoAnalyzerPreset** 和/或 **AudioAnalyzerPreset** 時，將您的媒體服務帳戶設定為 10 S3 媒體保留單元。

## <a name="does-v3-asset-have-no-assetfile-concept"></a>v3 資產是否沒有 AssetFile 概念？

AssetFile 已從 AMS API 中移除，以便讓媒體服務與儲存體 SDK 相依性分開。 現在是由儲存體 (而非媒體服務) 保留屬於儲存體的資訊。 

## <a name="where-did-client-side-storage-encryption-go"></a>用戶端儲存體加密怎麼不見了？

我們現在建議使用伺服器端儲存體加密 (預設為開啟狀態)。如需詳細資訊，請參閱[待用資料的 Azure 儲存體服務加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)。

## <a name="what-is-the-recommended-upload-method"></a>建議的上傳方法為何？

我們建議使用 HTTP(s) 內嵌。 如需詳細資訊，請參閱 [HTTP(s) 內嵌](job-input-from-http-how-to.md)。

## <a name="how-does-pagination-work"></a>分頁的運作方式為何？

媒體服務可對支援 OData 的資源支援 $top，但傳遞至 $top 的值必須小於 1000 (例如，分頁的頁面大小)。

這可讓您取得使用 $top 的小型項目範例 (例如，100 個最新項目)，或使用分頁功能逐頁查看所有項目。 

媒體服務不支援以使用者指定的頁面大小逐頁查看資料。

如需詳細資訊，請參閱[篩選、排序、分頁](assets-concept.md#filtering-ordering-paging)

## <a name="how-to-retrieve-an-entity-in-media-services-v3"></a>如何擷取媒體服務 v3 中的實體？

v3 是以統一 API 表面為基礎的，會公開建置於 **Azure Resource Manager** 上的管理和操作功能。 根據 **Azure Resource Manager**，資源名稱永遠是唯一的。 因此，您可以對資源名稱使用任何唯一識別碼字串 (例如，GUID)。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [媒體服務 v3 概觀](media-services-overview.md)
