---
title: 升級至 Azure 搜尋服務 .NET SDK 第 5 版 | Microsoft Docs
description: 升級至 Azure 搜尋服務 .NET SDK 第 5 版
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: brjohnst
ms.openlocfilehash: b08507d7685ce87a4c176385f750a72d6ae51ba3
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35766559"
---
# <a name="upgrading-to-the-azure-search-net-sdk-version-5"></a>升級至 Azure 搜尋服務 .NET SDK 第 5 版
如果您使用 4.0 預覽版或更舊版本的 [Azure 搜尋服務 .NET SDK](https://aka.ms/search-sdk)，本文會協助您將應用程式升級為使用第 5 版。

如需包括範例的 SDK 一般逐步解說，請參閱 [如何從 .NET 應用程式使用 Azure 搜尋服務](search-howto-dotnet-sdk.md)。

Azure 搜尋服務 .NET SDK 第 5 版包含了一些舊版所做的變更。 這些變更大部分是次要變更，所以變更程式碼應該只需要最少的工作。 請參閱 [升級步驟](#UpgradeSteps) 以取得如何變更您的程式碼以使用新的 SDK 版本的指示。

> [!NOTE]
> 如果您使用 2.0 預覽版或更舊版本，請先升級至第 3 版，再升級至第 5 版。 如需指示，請參閱[升級至 Azure 搜尋服務 .NET SDK 第 3 版](search-dotnet-sdk-migration.md)。
>
> 您的「Azure 搜尋服務」執行個體支援數個 REST API 版本，包括最新版本。 當一個版本不再是最新版本時，您仍可繼續使用該版本，但建議您將程式碼移轉成使用最新版本。 使用 REST API 時，您必須每個要求中透過 api-version 參數指定 API 版本。 使用 .NET SDK 時，您使用的 SDK 版本會決定對應的 REST API 版本。 如果您使用的是舊版 SDK，則即使服務已升級成支援新版 API 版本，您仍可繼續執行該程式碼而無須變更。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-5"></a>第 5 版的新功能
Azure 搜尋服務 .NET SDK 第 5 版以最新推出的 Azure 搜尋服務 REST API 版本為目標，也就是 2017-11-11。 這可讓您從 .NET 應用程式中使用 Azure 搜尋服務的新功能，包括：

* [同義字](search-synonyms.md)。
* 您現在可以透過程式設計方式來存取索引子執行歷程記錄中的警告 (請參閱 [.NET 參考](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexerexecutionresult?view=azure-dotnet)中 `IndexerExecutionResult` 的 `Warning` 屬性，以了解詳細資料)。
* .NET Core 2 的支援。
* 新的套件結構支援只使用所需的 SDK 組件 (請參閱[第 5 版的中斷性變更](#ListOfChanges)，以了解詳細資料)。

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升級步驟
首先，更新 `Microsoft.Azure.Search` 的 NuGet 參考，方法是使用 NuGet 封裝管理員主控台，或是在 Visual Studio 中用滑鼠右鍵按一下您的專案參考並選取 [管理 NuGet 封裝]。

一旦 NuGet 下載新的封裝和其相依性，請重建您的專案。 除非您使用不在新 GA SDK 中的預覽功能，否則應該會成功重建 (如果未成功，請在 [GitHub](https://github.com/azure/azure-sdk-for-net/issues) 上通知我們)。 如果是這樣，代表您已準備就緒！

請注意，由於 Azure 搜尋服務 .NET SDK 封裝的變更，您必須重建應用程式才能使用第 5 版。 [第 5 版的中斷性變更](#ListOfChanges)會詳述這些變更。

您可能會看到與過時的方法或屬性相關的其他建置警告。 警告將會包含要使用哪些內容取代過時功能的指示。 例如，如果您的應用程式使用 `IndexingParametersExtensions.DoNotFailOnUnsupportedContentType` 方法，您應該會收到警告，指出「此行為現在已預設啟用，因此已不再需要呼叫這個方法。」

一旦您修正所有編譯警告後，就可以變更您的應用程式，以視需要利用新的功能。 [第 5 版的新功能](#WhatsNew)會詳述 SDK 的新功能。

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-5"></a>第 5 版的中斷性變更
第 5 版最重要的中斷性變更是 `Microsoft.Azure.Search` 組件及其內容已分成四個不同的組件，現在會以四個個別的 NuGet 套件來散發：

 - `Microsoft.Azure.Search`：這是包含所有其他 Azure 搜尋服務相依套件的中繼套件。 如果您要從舊版 SDK 升級，只要升級此套件並重建，應該就能開始使用新版本。
 - `Microsoft.Azure.Search.Data`：如果您正在開發使用 Azure 搜尋服務的 .NET 應用程式，而且只需要查詢或更新索引中的文件，則請使用此套件。 如果您也需要建立或更新索引、同義字地圖或其他服務層級資源，請改用 `Microsoft.Azure.Search` 套件。
 - `Microsoft.Azure.Search.Service`：如果您正在開發 .NET 自動化來管理 Azure 搜尋服務的索引、同義字地圖、索引子、資料來源或其他服務層級資源，請使用此套件。 如果您只需要查詢或更新索引中的文件，請改用 `Microsoft.Azure.Search.Data` 套件。 如果您需要 Azure 搜尋服務的所有功能，請改用 `Microsoft.Azure.Search` 套件。
 - `Microsoft.Azure.Search.Common`：Azure 搜尋服務 .NET 程式庫所需的常用類型。 您不需要在應用程式中直接使用此套件；此套件乃是做為相依性之用。
 
這項變更在技術上造成中斷，因為在組件之間移動了許多類型。 這也是必須重建您的應用程式才能升級至第 5 版 SDK 的原因。

除了重建您的應用程式，第 5 版有少數幾項其他中斷性變更可能需要變更程式碼。

### <a name="removed-obsolete-members"></a>移除過時的成員

針對在舊版中標示為淘汰且在第 5 版移除的方法或屬性，您可能會看到相關的建置錯誤。 如果您遇到這類錯誤，解決方法如下︰

- 如果您使用 `IndexingParametersExtensions.IndexStorageMetadataOnly` 方法，請改用 `SetBlobExtractionMode(BlobExtractionMode.StorageMetadata)`。
- 如果您使用 `IndexingParametersExtensions.SkipContent` 方法，請改用 `SetBlobExtractionMode(BlobExtractionMode.AllMetadata)`。

### <a name="removed-preview-features"></a>已移除的預覽功能

如果您從 4.0 預覽版升級至第 5 版，請注意已移除 Blob 索引子的 JSON 陣列和 CSV 剖析支援，因為這些功能仍處於預覽階段。 具體而言，`IndexingParametersExtensions` 類別中已移除下列方法︰

- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

如果您的應用程式緊密相依於這些功能，便無法升級至 Azure 搜尋服務 .NET SDK 第 5 版。 您可以繼續使用 4.0 預覽版。 但請記住，**我們不建議在實際執行應用程式中使用預覽 SDK**。 預覽功能僅供評估，可能會變更。

## <a name="conclusion"></a>結論
如需更多有關使用 Azure 搜尋服務 .NET SDK 的詳細資料，請參閱 [.NET 做法](search-howto-dotnet-sdk.md)。

歡迎您提供 SDK 的意見反應。 如果您遇到問題，歡迎在 [Azure 搜尋服務 MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azuresearch)上尋求協助。 如果您發現錯誤，您可以在 [Azure .NET SDK GitHub 儲存機制](https://github.com/Azure/azure-sdk-for-net/issues)中提出問題。 請務必在問題的標題前面加上「[Azure 搜尋服務]」前置詞。

感謝您使用 Azure 搜尋服務！
