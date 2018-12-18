---
title: 升級至 Azure 搜尋服務 .NET 管理 SDK 第 2 版 | Microsoft Docs
description: 升級至 Azure 搜尋服務 .NET 管理 SDK 第 2 版
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 01/15/2018
ms.openlocfilehash: a6b6c01f0cc811abca90139e4c26c27b7bd7119f
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31790360"
---
# <a name="upgrading-to-the-azure-search-net-management-sdk-version-2"></a>升級至 Azure 搜尋服務 .NET 管理 SDK 第 2 版
如果您使用 1.0.2 版或更舊版本的 [Azure 搜尋服務 .NET 管理 SDK](https://aka.ms/search-mgmt-sdk)，本文會協助您將應用程式升級為使用第 2 版。

Azure 搜尋服務 .NET 管理 SDK 第 2 版包含一些對舊版所進行的變更。 這些變更大部分是次要變更，所以變更程式碼應該只需要最少的工作。 請參閱 [升級步驟](#UpgradeSteps) 以取得如何變更您的程式碼以使用新的 SDK 版本的指示。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-2"></a>第 2 版的新功能
Azure 搜尋服務 .NET 管理 SDK 第 2 版和舊版 SDK (特別是 2015-08-19 版) 一樣，都是以正式上市版的 Azure 搜尋服務管理 REST API 作為目標。 此 SDK 的變更完全是用戶端方面的變更，其目的是要改善 SDK 本身的可用性。 這些變更包括下列幾項：

* `Services.CreateOrUpdate` 和其非同步版本現在會自動輪詢佈建 `SearchService`，且在服務佈建完成之前不會傳回。 這可讓您不必自行撰寫這類輪詢程式碼。
* 如果您仍要手動輪詢服務佈建，則可以使用新的 `Services.BeginCreateOrUpdate` 方法，也可以使用它的其中一個非同步版本。
* 新方法 `Services.Update` 和其非同步版本都已新增至 SDK。 這些方法會使用 HTTP PATCH 來支援累加式的服務更新。 例如，您現在可以將僅包含所需 `partitionCount` 和 `replicaCount` 屬性 `SearchService` 的執行個體傳遞給這些方法，來調整服務。 目前仍支援呼叫 `Services.Get`、修改傳回的 `SearchService`，並將其傳遞至 `Services.CreateOrUpdate` 的舊方法，但您不再必須使用此方法。 

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升級步驟
首先，更新 `Microsoft.Azure.Management.Search` 的 NuGet 參考，方法是使用 NuGet 封裝管理員主控台，或是在 Visual Studio 中用滑鼠右鍵按一下您的專案參考並選取 [管理 NuGet 封裝]。

一旦 NuGet 下載新的封裝和其相依性，請重建您的專案。 根據您的程式碼結構情況，它可能會順利重新建置。 如果是這樣，代表您已準備就緒！

如果建置失敗，原因可能是您已實作某些已有所變更的 SDK 介面 (例如，為了進行單元測試)。 若要解決這個問題，您必須實作新的方法，例如 `BeginCreateOrUpdateWithHttpMessagesAsync`。

一旦您已修正任何建置錯誤，您就可以變更您的應用程式以視需要利用新的功能。 [第 2 版的新功能](#WhatsNew)會詳述 SDK 的新功能。

## <a name="conclusion"></a>結論
歡迎您提供 SDK 的意見反應。 如果您遇到問題，歡迎在 [Azure 搜尋服務 MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azuresearch)上尋求協助。 如果您發現錯誤，您可以在 [Azure .NET SDK GitHub 儲存機制](https://github.com/Azure/azure-sdk-for-net/issues)中提出問題。 請務必在問題的標題前面加上「[Azure 搜尋服務]」前置詞。

感謝您使用 Azure 搜尋服務！
