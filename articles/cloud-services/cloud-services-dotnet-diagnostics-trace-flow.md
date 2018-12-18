---
title: 使用 Azure 診斷追蹤雲端服務應用程式中的流程 | Microsoft Docs
description: 將追蹤訊息加入至 Azure 應用程式來協助偵錯、測量效能、監視、流量分析等等。
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 09934772-cc07-4fd2-ba88-b224ca192f8e
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/20/2016
ms.author: jeconnoc
ms.openlocfilehash: 2ba97e43616386a0ff8459316bfc4d3ddfe241a0
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39000890"
---
# <a name="trace-the-flow-of-a-cloud-services-application-with-azure-diagnostics"></a>使用 Azure 診斷追蹤雲端服務應用程式的流程
追蹤是一種方式，可讓您在應用程式執行時加以監視。 您可以使用 [System.Diagnostics.Trace](https://msdn.microsoft.com/library/system.diagnostics.trace.aspx)、[System.Diagnostics.Debug](https://msdn.microsoft.com/library/system.diagnostics.debug.aspx) 和 [System.Diagnostics.TraceSource](https://msdn.microsoft.com/library/system.diagnostics.tracesource.aspx) 類別，在記錄檔、文字檔或其他裝置中記錄錯誤和應用程式執行的相關資訊，供稍後分析。 如需追蹤的詳細資訊，請參閱 [追蹤和檢測應用程式](https://msdn.microsoft.com/library/zs6s4h68.aspx)。

## <a name="use-trace-statements-and-trace-switches"></a>使用追蹤陳述式和追蹤參數
藉由加入 [DiagnosticMonitorTraceListener](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.diagnostics.diagnosticmonitortracelistener.aspx) 至應用程式組態，並在您的應用程式程式碼中對 System.Diagnostics.Trace 或 System.Diagnostics.Debug 進行呼叫，藉此在雲端服務應用程式中實作追蹤。 將組態檔 app.config 用於背景工作角色，將 web.config 用於 Web 角色。 使用 Visual Studio 範本建立新的託管服務時，Azure 診斷會自動加入至專案，並且 DiagnosticMonitorTraceListener 會加入至您所加入角色的適當組態檔。

如需有關放置追蹤陳述式資訊，請參閱 [作法：加入 Trace 陳述式至應用程式程式碼](https://msdn.microsoft.com/library/zd83saa2.aspx)。

藉由在程式碼中放置 [Trace 參數](https://msdn.microsoft.com/library/3at424ac.aspx) ，您可以控制是否發生追蹤以及廣泛程度。 這可讓您監視您的應用程式在生產環境中的狀態。 對於在多部電腦上執行多個元件的商務應用程式來說，這特別重要。 如需詳細資訊，請參閱 [做法：設定追蹤參數](https://msdn.microsoft.com/library/t06xyy08.aspx)。

## <a name="configure-the-trace-listener-in-an-azure-application"></a>在 Azure 應用程式中設定追蹤接聽程式
Trace、Debug 和 TraceSource，需要您設定「接聽程式」來收集和記錄傳送的訊息。 接聽程式會收集、儲存和路由傳送追蹤訊息。 它們將追蹤輸出導向至適當的目標，例如記錄檔、視窗或文字檔。 Azure 診斷使用 [DiagnosticMonitorTraceListener](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.diagnostics.diagnosticmonitortracelistener.aspx) 類別。

完成下列程序之前，您必須初始化 Azure 診斷監視器。 若要這樣做，請參閱 [在 Microsoft Azure 中啟用診斷](cloud-services-dotnet-diagnostics.md)。

請注意，如果您使用 Visual Studio 所提供的範本，則會自動為您加入接聽程式的組態。

### <a name="add-a-trace-listener"></a>加入追蹤接聽程式
1. 開啟您的角色的 web.config 或 app.config 檔案。
2. 將下列程式碼新增至檔案。 變更 Version 屬性以使用您所參考的組件版本號碼。 除非有更新，否則組件版本不一定會隨著每個 Azure SDK 版本而變更。
   
    ```
    <system.diagnostics>
        <trace>
            <listeners>
                <add type="Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitorTraceListener,
                  Microsoft.WindowsAzure.Diagnostics,
                  Version=2.8.0.0,
                  Culture=neutral,
                  PublicKeyToken=31bf3856ad364e35"
                  name="AzureDiagnostics">
                    <filter type="" />
                </add>
            </listeners>
        </trace>
    </system.diagnostics>
    ```
   > [!IMPORTANT]
   > 請確定您有參照 Microsoft.WindowsAzure.Diagnostics 組件的專案參考。 更新上述 xml 中的版本號碼，以和參考的 Microsoft.WindowsAzure.Diagnostics 組件版本相符。
   > 
   > 
3. 儲存組態檔。

如需接聽程式的詳細資訊，請參閱 [追蹤接聽程式](https://msdn.microsoft.com/library/4y5y10s7.aspx)。

完成加入接聽程式的步驟之後，您可以加入您的追蹤陳述式到程式碼中。

### <a name="to-add-trace-statement-to-your-code"></a>將追蹤陳述式加入至您的程式碼
1. 開啟您的應用程式的原始程式檔。 例如，背景工作角色或 Web 角色的 <RoleName>.cs 檔案。
2. 加入下列 using 陳述式 (如果尚未加入)：
    ```
        using System.Diagnostics;
    ```
3. 在您想要用來擷取應用程式狀態資訊的位置加入 Trace 陳述式。 您可以使用各種方法來格式化 Trace 陳述式的輸出。 如需詳細資訊，請參閱 [做法：加入 Trace 陳述式至應用程式程式碼](https://msdn.microsoft.com/library/zd83saa2.aspx)。
4. 儲存原始程式檔。

