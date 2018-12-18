---
title: 為 Azure Application Insights 中的 Analytics 進行疑難排解 | Microsoft Docs
description: '有關於 Application Insights 分析的問題嗎？ 從這裡開始。 '
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 9bbd5859-3584-4d80-9b6d-d5910fa48baa
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2016
ms.author: mbullwin
ms.openlocfilehash: eeda0fa6ad8faa05baf0a9344e958d298fb80d8e
ms.sourcegitcommit: 828d8ef0ec47767d251355c2002ade13d1c162af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36936678"
---
# <a name="troubleshoot-analytics-in-application-insights"></a>疑難排解 Application Insights 中的分析
有關於 [Application Insights 分析](app-insights-analytics.md)的問題嗎？ 從這裡開始。 「分析」是強大的 Azure Application Insights 搜尋工具。

## <a name="limits"></a>限制
* 目前，查詢結果受限於僅只過去一週的資料。
* 我們測試的瀏覽器︰最新版本的 Chrome、Edge 及 Internet Explorer。

## <a name="known-incompatible-browser-extensions"></a>已知不相容的瀏覽器擴充功能
* Ghostery

停用擴充功能或使用不同的瀏覽器。

## <a name="e-a"></a> 「未預期的錯誤」
![[未預期的錯誤] 畫面](./media/app-insights-analytics-troubleshooting/010.png)

在入口網站執行階段期間發生的內部錯誤 – 未處理的例外狀況。

* 清除瀏覽器的快取。 

## <a name="e-b"></a>403 ... 請嘗試重新載入
![403 ... 請嘗試重新載入](./media/app-insights-analytics-troubleshooting/020.png)

發生驗證相關的錯誤 (在驗證期間或存取權杖產生期間)。 若未變更瀏覽器設定，入口網站可能無法復原。

* 確認已在瀏覽器中 [啟用協力廠商 Cookie](#cookies) 。 

## <a name="authentication"></a>403 ... 確認安全性區域
![403 ... 確認安全性區域](./media/app-insights-analytics-troubleshooting/030.png)

發生驗證相關的錯誤 (在驗證期間或存取權杖產生期間)。 若未變更瀏覽器設定，入口網站可能無法復原。

1. 確認已在瀏覽器中 [啟用協力廠商 Cookie](#cookies) 。 
2. 您使用我的最愛、書籤或儲存的連結來開啟分析入口網站嗎？ 您登入時所使用的認證與您儲存連結時使用的認證不同嗎？
3. 嘗試使用 InPrivate/incognito 瀏覽器視窗 (關閉所有這類視窗之後)。 您必須提供認證。 
4. 開啟另一個 (一般) 瀏覽器視窗並前往 [Azure](https://portal.azure.com)。 登出。然後開啟您的連結並使用正確的認證登入。
5. Edge 和 Internet Explorer 的使用者也會在不支援信任的區域設定時收到此錯誤。
   
    確認[分析入口網站](https://portal.azure.com)和 [Azure Active Directory 入口網站](https://portal.azure.com)位於相同的安全性區域：
   
   * 在 Internet Explorer 中，開啟 [網際網路選項]、[安全性]、[信任的網站]、[網站]：
     
     ![[網際網路選項] 對話方塊，將網站新增到信任的網站](./media/app-insights-analytics-troubleshooting/033.png)
     
     在 [網站] 清單中，如果包含下列任一個 URL，請確定也會包含其他的 URL：
     
     https://analytics.applicationinsights.io<br/>
     https://login.microsoftonline.com<br/>
     https://login.windows.net

## <a name="e-d"></a>404 ...找不到資源
![404 ... 找不到資源](./media/app-insights-analytics-troubleshooting/040.png)

應用程式資源已從 Application Insights 中刪除，且不再提供。 這可能會在您將 URL 儲存到 [分析] 頁面時發生。

## <a name="e-e"></a>403 ...沒有授權
![403 ... 未獲授權](./media/app-insights-analytics-troubleshooting/050.png)

您沒有權限可在分析中開啟此應用程式。

* 您的連結是由其他人所提供嗎？ 要求他們確認您是 [這個資源群組的讀取者或參與者](app-insights-resources-roles-access-control.md)。
* 您使用不同的認證來儲存連結嗎？ 開啟 [Azure 入口網站](https://portal.azure.com)、登出，然後提供正確的認證再試一次連結。

## <a name="html-storage"></a>403 ...HTML5 儲存體
我們的入口網站會使用 HTML5 localStorage 和 sessionStorage。

* Chrome︰設定、隱私權、內容設定。
* Internet Explorer︰網際網路選項、進階索引標籤、安全性、啟用 DOM 儲存

![403 ... 嘗試啟用 HTML5 儲存體](./media/app-insights-analytics-troubleshooting/060.png)

## <a name="e-g"></a>404 ...找不到訂用帳戶
![404 ...找不到訂用帳戶](./media/app-insights-analytics-troubleshooting/070.png)

URL 無效。 

* 開啟 [Application Insights 入口網站](https://portal.azure.com)中的應用程式資源。 然後使用 [分析] 按鈕。

## <a name="e-h"></a>404 ... 頁面不存在
![404 ...頁面不存在](./media/app-insights-analytics-troubleshooting/080.png)

URL 無效。

* 開啟 [Application Insights 入口網站](https://portal.azure.com)中的應用程式資源。 然後使用 [分析] 按鈕。

## <a name="cookies"></a>啟用協力廠商 cookie
  請參閱 [如何停用協力廠商 Cookie](http://www.digitalcitizen.life/how-disable-third-party-cookies-all-major-browsers)，但請注意，我們需要 **啟用** 它們。


[!INCLUDE [app-insights-analytics-footer](../../includes/app-insights-analytics-footer.md)]

