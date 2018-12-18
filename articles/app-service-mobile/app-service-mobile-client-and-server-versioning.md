---
title: Mobile Apps 和行動服務中的用戶端和伺服器 SDK 版本控制 | Microsoft Docs
description: 列出適用於行動服務和 Azure Mobile Apps 的用戶端 SDK 和與伺服器 SDK 版本的相容性
services: app-service\mobile
documentationcenter: ''
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 35b19672-c9d6-49b5-b405-a6dcd1107cd5
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 37bf36af535eb9b5c8b0ba38434b71f1a6686811
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "27593369"
---
# <a name="client-and-server-versioning-in-mobile-apps-and-mobile-services"></a>Mobile Apps 和行動服務中的用戶端和伺服器版本控制
Azure 行動服務的最新版本是 Azure App Service 的 **Mobile Apps** 功能。

Mobile Apps 用戶端和伺服器 SDK 最初是以行動服務中的 SDK 為基礎，但是它們彼此「不」相容。
也就是說，您必須使用 Mobile Apps 用戶端 SDK 搭配 Mobile Apps 伺服器 SDK，「行動服務」也是類似作法。 您可以透過用戶端和伺服器 SDK 所使用的特殊標頭值 `ZUMO-API-VERSION` 來強制執行。

注意：每當這份文件提及 *行動服務* 後端時，它不一定需要裝載於行動服務。 現在可以將行動服務移轉到 App Service 上執行而不用變更任何程式碼，但是服務還是會使用「行動服務」SDK 版本。

若要深入了解如何移轉至 App Service 而不變更任何程式碼，請參閱 [將行動服務移轉至 Azure App Service]一文。

## <a name="header-specification"></a>標頭規格
鍵值 `ZUMO-API-VERSION` 可以在 HTTP 標頭或查詢字串中指定。 值是版本字串，格式為 **x.y.z**。

例如︰

GET https://service.azurewebsites.net/tables/TodoItem

HEADERS: ZUMO-API-VERSION: 2.0.0

POST https://service.azurewebsites.net/tables/TodoItem?ZUMO-API-VERSION=2.0.0

## <a name="opting-out-of-version-checking"></a>選擇不進行版本檢查
您可以將應用程式設定 **MS_SkipVersionCheck** 的值設定為 **true**，選擇不要進行版本檢查。 在 web.config 中或在 Azure 入口網站的 [應用程式設定] 區段中都可指定這項設定。

> [!NOTE]
> 行動服務和 Mobile Apps 之間有許多行為改變了，尤其是在離線同步處理、驗證和推播通知的區域。 您應該在完成測試之後才選擇不要進行版本檢查，確保這些行為的變更不會影響您的 app 功能。
>
>

## <a name="summary-of-compatibility-for-all-versions"></a>所有版本的相容性摘要
下圖顯示所有的用戶端和伺服器類型之間的相容性。 後端是根據所使用的伺服器 SDK，而分類為行動**服務**或 **Mobile Apps**。

|  | **行動服務** Node.js 或 .NET | **Mobile Apps** Node.js 或 .NET |
| --- | --- | --- |
| [行動服務用戶端] |確定 |錯誤\* |
| [Mobile Apps 用戶端] |錯誤\* |確定 |

\*這可以藉由指定 **MS_SkipVersionCheck** 來控制。

<!-- IMPORTANT!  The anchors for Mobile Services and Mobile Apps MUST be 1.0.0 and 2.0.0 respectively, since there is an exception error message that uses those anchors. -->

<!-- NOTE: the fwlink to this document is http://go.microsoft.com/fwlink/?LinkID=690568 -->

## <a name="1.0.0"></a>行動服務用戶端和伺服器
下表中的用戶端 SDK 可與 **行動服務**相容。

注意：行動服務用戶端 SDK「不會」傳送 `ZUMO-API-VERSION` 的標頭值。 如果服務收到此標頭或查詢字串值，將會傳回錯誤，除非您如上面所述明確選擇不要進行檢查。

### <a name="MobileServicesClients"></a>行動「服務」用戶端 SDK
| 用戶端平台 | 版本 | 版本標頭值 |
| --- | --- | --- |
| 受控用戶端 (Windows、Xamarin) |[1.3.2](https://www.nuget.org/packages/WindowsAzure.MobileServices/1.3.2) |n/a |
| iOS |[2.2.2](http://aka.ms/gc6fex) |n/a |
| Android |[2.0.3](https://go.microsoft.com/fwLink/?LinkID=280126) |n/a |
| HTML |[1.2.7](http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.2.7.min.js) |n/a |

### <a name="mobile-services-server-sdks"></a>行動「服務」  伺服器 SDK
| 伺服器平台 | 版本 | 接受的版本標頭 |
| --- | --- | --- |
| .NET |[WindowsAzure.MobileServices.Backend.* 版本 1.0.x](https://www.nuget.org/packages/WindowsAzure.MobileServices.Backend/) |**無版本標頭** |
| Node.js |(敬請期待) |**無版本標頭** |

<!-- TODO: add Node npm version -->

### <a name="behavior-of-mobile-services-backends"></a>行動服務後端的行為
| ZUMO-API-VERSION | MS_SkipVersionCheck 的值 | Response |
| --- | --- | --- |
| 未指定 |任意 |200 - 確定 |
| 任何值 |True |200 - 確定 |
| 任何值 |False/未指定 |400 - 不正確的要求 |

## <a name="2.0.0"></a>Azure Mobile Apps 用戶端和伺服器
### <a name="MobileAppsClients"></a> Mobile Apps 用戶端 SDK
版本檢查是從 **Azure Mobile Apps**下列版本的用戶端 SDK 開始引進：

| 用戶端平台 | 版本 | 版本標頭值 |
| --- | --- | --- |
| 受控用戶端 (Windows、Xamarin) |[2.0.0](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/2.0.0) |2.0.0 |
| iOS |[3.0.0](http://go.microsoft.com/fwlink/?LinkID=529823) |2.0.0 |
| Android |[3.0.0](http://go.microsoft.com/fwlink/?LinkID=717033&clcid=0x409) |3.0.0 |

<!-- TODO: add HTML version when released -->

### <a name="mobile-apps-server-sdks"></a>Mobile「Apps」  伺服器 SDK
下列伺服器 SDK 版本包含版本檢查：

| 伺服器平台 | SDK | 接受的版本標頭 |
| --- | --- | --- |
| .NET |[Microsoft.Azure.Mobile.Server](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/) |2.0.0 |
| Node.js |[azure-mobile-apps)](https://www.npmjs.com/package/azure-mobile-apps) |2.0.0 |

### <a name="behavior-of-mobile-apps-backends"></a>Mobile Apps 後端的行為
| ZUMO-API-VERSION | MS_SkipVersionCheck 的值 | Response |
| --- | --- | --- |
| x.y.z 或 Null |True |200 - 確定 |
| Null |False/未指定 |400 - 不正確的要求 |
| 1.x.y |False/未指定 |400 - 不正確的要求 |
| 2.0.0-2.x.y |False/未指定 |200 - 確定 |
| 3.0.0-3.x.y |False/未指定 |400 - 不正確的要求 |

## <a name="next-steps"></a>後續步驟
* [將行動服務移轉至 Azure App Service]

[行動服務用戶端]: #MobileServicesClients
[Mobile Apps 用戶端]: #MobileAppsClients


[Mobile App Server SDK]: http://www.nuget.org/packages/microsoft.azure.mobile.server
[將行動服務移轉至 Azure App Service]: app-service-mobile-migrating-from-mobile-services.md
