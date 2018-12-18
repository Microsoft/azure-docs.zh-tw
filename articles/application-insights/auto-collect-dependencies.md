---
title: Azure Application Insights - 相依性自動收集 | Microsoft Docs
description: Application Insights 會自動收集相依項目，並將其視覺化
services: application-insights
documentationcenter: .net
author: nikmd23
manager: carmonm
ms.service: application-insights
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: reference
ms.date: 08/13/2018
ms.reviewer: mbullwin
ms.author: nimolnar
ms.openlocfilehash: f20963f030c9040b696f7d6a33b25bcee2dc517f
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "40130270"
---
# <a name="dependency-auto-collection"></a>相依性自動收集

以下是目前支援的相依性呼叫清單，系統會自動將這些呼叫偵測為相依性，而且不需要修改應用程式的任何其他程式碼。 這包含對通訊程式庫、儲存體用戶端、記錄和計量程式庫執行的傳出呼叫，以及傳入應用程式架構和伺服器的傳入呼叫。 這些相依項目會在 Application Insights 中的[應用程式對應](https://docs.microsoft.com/azure/application-insights/app-insights-app-map)和[交易診斷](https://docs.microsoft.com/azure/application-insights/app-insights-transaction-diagnostics)檢視中加以視覺化。 如果您的相依性不在以下清單中，您仍可以透過[追蹤相依性呼叫](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackdependency)，以手動方式進行追蹤。

## <a name="net"></a>.NET

| 應用程式架構| 版本 |
| ------------------------|----------|
| ASP.NET WebForm | 4.5+ |
| ASP.NET MVC | 4+ |
| ASP.NET WebAPI | 4.5+ |
| ASP.NET Core | 1.1+ |
| <b> 通訊程式庫</b> |
| [HttpClient](https://www.microsoft.com/net/) | 4.5+、.NET Core 1.1+ |
| [SqlClient](https://www.nuget.org/packages/System.Data.SqlClient) | .NET Core 1.0+、NuGet 4.3.0 |
| [EventHubs 用戶端 SDK](https://www.nuget.org/packages/Microsoft.Azure.EventHubs) | 1.1.0 |
| [ServiceBus 用戶端 SDK](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus) | 3.0.0 |
| <b>儲存體用戶端</b>|  |
| ADO.NET | 4.5+ |
| <b>記錄程式庫</b> |  |
| ILogger | 1.1+ |
| System.Diagnostics.Trace | 4.5+ |
| [nLog](https://www.nuget.org/packages/NLog/) | 4.4.12+ |
| [log4net](https://www.nuget.org/packages/log4net/) | NetStandard 1.3 上的 2.0.8+、.NET 4.5+ 上的 2.0.6+ |

## <a name="java"></a>Java
| 應用程式伺服器 | 版本 |
|-------------|----------|
| [Tomcat](https://tomcat.apache.org/) | 7、8 | 
| [JBoss EAP](https://developers.redhat.com/products/eap/download/) | 6、7 |
| [Jetty](http://www.eclipse.org/jetty/) | 9 |
| <b>應用程式架構</b> |  |
| [Spring](https://spring.io/) | 3.0 |
| [Spring Boot](https://spring.io/projects/spring-boot) | 1.5.9+<sup>*</sup> |
| Java Servlet | 3.1+ |
| <b>通訊程式庫</b> |  |
| [Apache HTTP 用戶端](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient) | 4.3+<sup>†</sup> |
| <b>儲存體用戶端</b> | |
| [SQL Server]( https://mvnrepository.com/artifact/com.microsoft.sqlserver/mssql-jdbc) | 1+<sup>†</sup> |
| [Oracle]( http://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html) | 1+<sup>†</sup> |
| [MySql]( https://mvnrepository.com/artifact/mysql/mysql-connector-java) | 1+<sup>†</sup> |
| <b>記錄程式庫</b> | |
| [Logback](https://logback.qos.ch/) | 1+ |
| [Log4j](https://logging.apache.org/log4j/) | 1.2+ |
| <b>計量程式庫</b> |  |
| JMX | 1.0+ |

> [!NOTE]
> *不包含回應式程式設計支援。
> <br>†需要安裝 [JVM 代理程式](https://docs.microsoft.com/azure/application-insights/app-insights-java-agent#install-the-application-insights-agent-for-java)。

## <a name="nodejs"></a>Node.js

| 通訊程式庫 | 版本 |
| ------------------------|----------|
| [HTTP](https://nodejs.org/api/http.html)、[HTTPS](https://nodejs.org/api/https.html) | 0.10+ |
| <b>儲存體用戶端</b> | |
| [Redis](https://www.npmjs.com/package/redis) | 2.x |
| [MongoDb](https://www.npmjs.com/package/mongodb)、[MongoDb Core](https://www.npmjs.com/package/mongodb-core) | 2.0.0 - 2.3.0 |
| [MySQL](https://www.npmjs.com/package/mysql) | 2.0.0 - 2.14.x |
| [PostgreSql](https://www.npmjs.com/package/pg); | 6.x |
| [pg-pool](https://www.npmjs.com/package/pg-pool) | 1.x |
| <b>記錄程式庫</b> | |
| [console](https://nodejs.org/api/console.html) | 0.10+ |
| [Bunyan](https://www.npmjs.com/package/bunyan) | 1.x |
| [Winston](https://www.npmjs.com/package/winston) | 2.x |

## <a name="javascript"></a>JavaScript

| 通訊程式庫 | 版本 |
| ------------------------|----------|
| [XMLHttpRequest](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) | 全部 |

## <a name="next-steps"></a>後續步驟

- 設定 [.NET](app-insights-asp-net-dependencies.md) 的自訂相依性追蹤。
- 設定 [Java](app-insights-java-agent.md) 的自訂相依性追蹤。
- [撰寫自訂相依性遙測](app-insights-api-custom-events-metrics.md#trackdependency)
- 如需 Application Insights 類型和資料模型，請參閱[資料模型](application-insights-data-model.md)。
- 查看 Application Insights 支援的[平台](app-insights-platforms.md)。
