---
title: 升級至最新的 Azure 搜尋服務 REST API 版本 | Microsoft Docs
description: 升級至最新的 Azure 搜尋服務 REST API 版本
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: brjohnst
ms.openlocfilehash: 2efe7769f68988f3c0d52c8806b78c1b96d8c639
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39620224"
---
# <a name="upgrading-to-the-latest-azure-search-service-rest-api-version"></a>升級至最新的 Azure 搜尋服務 REST API 版本
如果您正在使用舊版的 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/)，本文將協助您升級應用程式，以使用正式推出的最新 API 版本 2017-11-11。

REST API 的版本 2017-11-11 包含一些較早版本的變更。 這些是大部分具有回溯相容性，因此變更程式碼應該只需要最少的工作 (視您之前使用的版本而定)。 請參閱[升級步驟](#UpgradeSteps)以取得如何變更您的程式碼以使用新的 API 版本的指示。

> [!NOTE]
> 您的「Azure 搜尋服務」執行個體支援數個 REST API 版本，包括最新版本。 當一個版本不再是最新版本時，您仍可繼續使用該版本，但建議您將程式碼移轉成使用最新版本。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-2017-11-11"></a>版本 2017-11-11 的新功能
版本 2017-11-11 是 Azure 搜尋服務 REST API 的最新正式推出版本。 這個 API 版本的新功能包括︰

* [同義字](search-synonyms.md)。 新的同義字功能可讓您定義對等的詞彙並擴大查詢的範圍。
* [支援有效率地編製文字 Blob 的索引](https://docs.microsoft.com/azure/search/search-howto-indexing-azure-blob-storage#IndexingPlainText)。 Azure Blob 索引子的新 `text` 剖析模式大幅提升編製索引效能。
* [服務統計資料 API](https://docs.microsoft.com/rest/api/searchservice/get-service-statistics)。 使用這個新的 API 檢視資源在 Azure 搜尋服務中的目前使用量和限制。

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升級步驟
如果您從 GA 版本 2015-02-28 或 2016-09-01 升級，則除了變更版本號碼之外，可能不需要對您的程式碼進行任何變更。 您可能需要變更程式碼的唯一情況是︰

* 您的程式碼在 API 回應中傳回無法辨認的屬性時失敗。 您的應用程式預設應該會略過不了解的屬性。
* 您的程式碼仍然存在 API 要求，並嘗試將它們重新傳送給新的 API 版本。 例如，發生原因可能是您的應用程式持續保存搜尋服務 API 所傳回的接續 Token (如需詳細資訊，請在[搜尋服務 API 參考](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)) 中查找 `@search.nextPageParameters`。

如果下列其中一種情況適用您的情況，您可能需要據此變更程式碼。 否則，除非您想要開始使用版本 2017-11-11 的[新功能](#WhatsNew)，否則應該不需要進行任何變更。

如果您要從預覽 API 版本升級，則也會套用上述作業，但也必須知道版本 2017-11-11 中未提供某些預覽功能：

* Azure Blob 儲存體索引子支援包含 JSON 陣列中的 CSV 檔案和 Blob。
* "More like this" 查詢

如果您的程式碼使用這些功能，則升級至 2017-11-11 時需要移除它們的使用。

> [!IMPORTANT]
> 請記住，預覽 API 是針對測試與評估，不應該用於生產環境。
> 
> 

## <a name="conclusion"></a>結論
如果您需要使用 Azure 搜尋服務 REST API 的更多詳細資料，請參閱 MSDN 上最近更新的 [API 參考](https://docs.microsoft.com/rest/api/searchservice/)。

歡迎您提供 Azure 搜尋服務的意見反應。 如果您遇到問題，歡迎在 [Azure 搜尋服務 MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azuresearch) 或 [StackOverflow](http://stackoverflow.com/)上尋求協助。 如果您要在 StackOverflow 上詢問 Azure 搜尋服務問題，請一定要將它標上 `azure-search`。

感謝您使用 Azure 搜尋服務！

