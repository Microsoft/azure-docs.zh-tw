---
title: Application Insights：程式語言、平台和整合 | Microsoft Docs
description: 適用於 Application Insights 的程式語言、平台和整合
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 974db106-54ff-4318-9f8b-f7b3a869e536
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 09/01/2016
ms.reviewer: olegan
ms.author: mbullwin
ms.openlocfilehash: 4f474ad234c80a0dcb5a9f704a263a97e7df0cc1
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39174229"
---
# <a name="developer-analytics-languages-platforms-and-integrations"></a>開發人員分析：程式語言、平台和整合
這些項目是我們曾聽過的 [Application Insights](app-insights-overview.md) 實作，包括一些協力廠商提供的實作。

## <a name="languages---officially-supported-by-application-insights-team"></a>程式語言 - Application Insights 團隊正式支援
* [C#|VB (.NET)](app-insights-asp-net.md)
* [Java](app-insights-java-get-started.md)
* [JavaScript web pages](app-insights-javascript.md)
* [Node.JS](app-insights-nodejs.md)

## <a name="languages---community-supported"></a>程式語言 - 社群支援
* [F#](https://safe-stack.github.io/docs/template-azure-ai/)
* [PHP](https://github.com/Microsoft/ApplicationInsights-PHP)
* [Python](https://pypi.python.org/pypi/applicationinsights/0.1.0)
* [Ruby](https://rubygems.org/gems/application_insights)
* [任何其他項目](#projects)

## <a name="platforms-and-frameworks"></a>平台和架構
* [ASP.NET](app-insights-asp-net.md)
* [ASP.NET - 適用於即時應用程式](app-insights-monitor-performance-live-website-now.md)
* [ASP.NET Core](app-insights-asp-net-core.md)
* [Android](app-insights-mobile-center-quickstart.md) (App Center)
* [Android](https://github.com/Microsoft/ApplicationInsights-Android) (App Center)
* [Angular](https://github.com/MarkPieszak/angular-application-insights)
* [Azure Web Apps](app-insights-azure-web-apps.md)
* [Azure 雲端服務](app-insights-cloudservices.md)&#151;同時包含 Web 與背景工作角色
* [Azure Functions](https://github.com/christopheranderson/azure-functions-app-insights-sample)
* [Docker](app-insights-docker.md)
* [Glimpse](https://azure.microsoft.com/blog/glimpse-application-insights/)
* [iOS](app-insights-mobile-center-quickstart.md) (App Center)
* [Ionic](https://github.com/SoftwarePioniere/ionic-application-insights)
* [iOS](https://github.com/Microsoft/ApplicationInsights-iOS) (App Center)
* [J2EE](app-insights-java-get-started.md)
* [J2EE - 適用於即時應用程式](app-insights-java-live.md)
* [Node.JS](https://www.npmjs.com/package/applicationinsights)
* [OSX](https://github.com/Microsoft/ApplicationInsights-OSX)
* [SAFE 堆疊](https://safe-stack.github.io/docs/template-azure-ai/)
* [Spring](http://joe.blog.freemansoft.com/2015/12/enabling-microsoft-application-insight.html)
* [通用 Windows app](app-insights-mobile-center-quickstart.md) (App Center)
* [WCF](https://github.com/Microsoft/ApplicationInsights-SDK-Labs/blob/master/WCF/readme.md)
* [Windows 桌面應用程式、服務和背景工作角色](app-insights-windows-desktop.md)
* [任何其他項目](#projects)

## <a name="logging-frameworks"></a>記錄架構
* [Log4Net、NLog 或 System.Diagnostics.Trace](app-insights-asp-net-trace-logs.md)
* [Java、Log4J 或 Logback](app-insights-java-trace-logs.md)
* [語意記錄 (SLAB)](https://github.com/fidmor89/SLAB_AppInsights) - 與[語意記錄應用程式區塊](https://msdn.microsoft.com/library/dn440729.aspx)整合
* [雲端式負載測試](http://blogs.msdn.com/b/visualstudioalm/archive/2015/07/30/getting-application-insights-counters-with-cloud-based-load-testing.aspx)
* [LogStash 外掛程式](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-output-applicationinsights)
* [Log Analytics](https://blogs.technet.microsoft.com/msoms/2016/09/26/application-insights-connector-in-oms/)
* [Logary](https://www.nuget.org/packages/Logary.Targets.AppInsights/)
* [Logrus](https://github.com/jjcollinge/logrus-appinsights)

## <a name="content-management-systems"></a>內容管理系統
* [Concrete](https://github.com/fidmor89/appInsights-Concrete)
* [Drupal](https://github.com/fidmor89/AppInsights-Drupal)
* [Joomla](https://github.com/fidmor89/AppInsights-Joomla)
* [Orchard](https://azure.microsoft.com/blog/integrating-application-insights-into-a-modular-cms-and-a-multi-tenant-public-saas/preview/)
* [SharePoint](app-insights-sharepoint.md)
* [WordPress](https://wordpress.org/plugins/application-insights/)

## <a name="export-and-data-analysis"></a>匯出和資料分析
* [Alooma](https://www.alooma.com/blog/application-insights-amazon-redshift)
* [Power BI](http://blogs.msdn.com/b/powerbi/archive/2015/11/04/explore-your-application-insights-data-with-power-bi.aspx)
* [串流分析](app-insights-export-power-bi.md)

## <a name="projects"></a> 建置您自己的 SDK
如果還沒有適合您程式語言或平台使用的 SDK，您會想要建置一個嗎？ 查看 [GitHub 上之 Application Insights SDK 專案](https://github.com/Microsoft/AppInsights-Home)中列出的現有 SDK 程式碼。
