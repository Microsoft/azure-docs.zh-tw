---
title: Azure 搜尋服務的 API 版本 | Microsoft Docs
description: Azure 搜尋服務 REST API 與 .NET SDK 中用戶端程式庫的版本原則。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 06/28/2018
ms.author: brjohnst
ms.openlocfilehash: 8d1e30b0bca3c63fe4528c06e5389d8cbe27a7e6
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37113600"
---
# <a name="api-versions-in-azure-search"></a>Azure 搜尋服務中的 API 版本
「Azure 搜尋服務」會定期推出功能更新。 有時候 (但並不一定)，這些更新會需要新版本的 API，以維持回溯相容性。 發行新版本可讓您控制在程式碼中整合搜尋服務更新的時機與方式。

因此，Azure 搜尋服務小組只在必要時發佈新版本，因為這牽涉到需要花費一些心力升級您的程式碼以使用新的 API 版本。 只有在以破壞回溯相容性的方式變更 API 的某些層面時，才需要新版本。 在修正現有的功能，或新功能會變更現有 API 的介面區時，就會發生這類變更。

相同的規則適用於 SDK 更新。 Azure 搜尋服務 SDK 會遵循 [semantic versioning (語意化版本控制系統)](http://semver.org/) 規則，這代表其版本具有三個部分：主要、次要和組建編號 (例如 1.1.0)。 只有在變更會破壞回溯相容性時，才會發行新的主要版本 SDK。 非重大功能更新會遞增次要版本，而錯誤修正只會遞增組建版本。

> [!NOTE]
> 您的「Azure 搜尋服務」執行個體支援數個 REST API 版本，包括最新版本。 當一個版本不再是最新版本時，您仍可繼續使用該版本，但建議您將程式碼移轉成使用最新版本。 使用 REST API 時，您必須每個要求中透過 api-version 參數指定 API 版本。 使用 .NET SDK 時，您使用的 SDK 版本會決定對應的 REST API 版本。 如果您使用的是舊版 SDK，則即使服務已升級成支援新版 API 版本，您仍可繼續執行該程式碼而無須變更。

## <a name="snapshot-of-current-versions"></a>目前版本的快照
以下是 Azure 搜尋服務所有程式設計介面目前版本的快照。

| 介面 | 最新主要版本 | 狀態 |
| --- | --- | --- |
| [.NET SDK](https://aka.ms/search-sdk) |5.0 |正式推出，2018 年 4 月發行 |
| [.NET SDK 預覽版](https://aka.ms/search-sdk-preview) |4.0.1-preview |預覽版，2017 年 5 月發行 |
| [服務 REST API](https://docs.microsoft.com/rest/api/searchservice/) |2017-11-11 |正式推出 |
| [服務 REST API 2017-11-11-Preview](search-api-2017-11-11-preview.md) |2017-11-11-Preview |預覽 |
| [.NET 管理 SDK](https://aka.ms/search-mgmt-sdk) |2.0 |正式推出 |
| [管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/) |2015-08-19 |正式推出 |

針對 REST API，必須在每個呼叫中納入 `api-version` 。 使用 `api-version` 會使將特定版本 (例如預覽 API) 作為目標較為容易。 下列範例說明如何指定 `api-version` 參數：

    GET https://adventure-works.search.windows.net/indexes/bikes?api-version=2017-11-11

> [!NOTE]
> 雖然每個要求都具有 `api-version`，我們仍建議您針對所有的 API 要求使用相同的版本。 特別是當新的 API 版本引入舊版本無法辨識的屬性或作業時更建議您這樣做。 混合的 API 版本可能會有非預期的結果，應該予以避免。
>
> 服務 REST API 和管理 REST API 的版本是彼此獨立的。 版本號碼如有類似，純屬巧合。

正式推出 (或 GA) 的 API 可以用於實際執行，並且受到 Azure 服務等級協定所約束。 預覽版本具有實驗性功能，那些功能並不會都移轉到 GA 版本上。 **強烈建議您避免在實際執行的應用程式中使用預覽 API。**

## <a name="about-preview-and-generally-available-versions"></a>關於預覽與正式推出版本
Azure 搜尋服務一律會先透過 REST API 預先推出實驗性功能，然後再透過 .NET SDK 的發行前版本推出。

預覽功能適用於測試和實驗，目標為收集有關功能設計和實作的意見反應。 基於這個原因，預覽功能可能會隨時間變更，很可能以中斷回溯相容性的方式進行。 這與 GA 版本中的功能形成鮮明對比，除了小型的回溯相容性修正和增強功能外，這些功能很穩定，不太可能變更。 此外，預覽功能不一定最終都會納入 GA 版本。

基於這些原因，建議您避免撰寫依存於預覽版本的實際程式碼。 如果您正在使用舊版預覽版本，建議您移轉到正式推出的 (GA) 版本。

如需程式碼移轉的 .NET SDK：指南，請參閱 [升級 .NET SDK](search-dotnet-sdk-migration.md)。

正式推出意指 Azure 搜尋服務已受到服務等級協定 (SLA) 約束。 SLA 可以在 [Azure 搜尋服務等級協定](https://azure.microsoft.com/support/legal/sla/search/v1_0/)中找到。
