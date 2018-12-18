---
title: 處理雲端服務生命週期事件 | Microsoft Docs
description: 了解如何在 .NET 中使用雲端服務角色的生命週期方法
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 39b30acd-57b9-48b7-a7c4-40ea3430e451
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.author: jeconnoc
ms.openlocfilehash: 56f7b5e3b303ce68868f15528d1ec200919b52aa
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001553"
---
# <a name="customize-the-lifecycle-of-a-web-or-worker-role-in-net"></a>在 .NET 中自訂 Web 或背景工作角色的生命週期
當您建立背景工作角色時，擴充可為您提供覆寫方法並讓您回應生命週期事件的 [RoleEntryPoint](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx) 類別。 若是 Web 角色，此類別是選擇性的，因此您必須使用它來回應生命週期事件。

## <a name="extend-the-roleentrypoint-class"></a>擴充 RoleEntryPoint 類別
[RoleEntryPoint](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx) 類別包含 Azure 在**啟動**、**執行**或**停止** Web 或背景工作角色時所呼叫的方法。 您可以選擇性地覆寫這些方法來管理角色初始化、角色關機順序或角色的執行緒。 

擴充 **RoleEntryPoint**時，您應該注意下列方法的行為：

* [OnStart](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx) 和 [OnStop](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx) 方法會傳回布林值，因此可能會從這些方法傳回 **false**。
  
   如果您的程式碼傳回 **false**，會突然終止角色處理序，而不會執行您既有的任何關機順序。 一般而言，您應該避免從 **OnStart** 方法傳回 **false**。
* **RoleEntryPoint** 方法多載中未能攔截的任何例外狀況，都會被視為未處理的例外狀況。
  
   如果在其中一個生命週期方法內發生例外狀況，Azure 將會引發 [UnhandledException](https://msdn.microsoft.com/library/system.appdomain.unhandledexception.aspx) 事件，然後處理序便會終止。 您的角色離線之後，Azure 將重新啟動該角色。 當未處理的例外狀況發生時，不會引發 [Stopping](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx) 事件，也不會呼叫 **OnStop** 方法。

如果您的角色無法啟動，或在初始化、忙碌和停止狀態之間循環，每次角色重新啟動時，您的程式碼可能會在其中一個生命週期事件內擲回未處理的例外狀況。 在此情況下，使用 [UnhandledException](https://msdn.microsoft.com/library/system.appdomain.unhandledexception.aspx) 事件判斷造成此例外狀況的原因，並適當地處理。 您的角色可能也會從 [Run](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法傳回，而導致該角色重新啟動。 如需有關部署狀態的詳細資訊，請參閱 [導致角色循環的常見問題](cloud-services-troubleshoot-common-issues-which-cause-roles-recycle.md)。

> [!NOTE]
> 如果您使用 **Azure Tools for Microsoft Visual Studio** 開發您的應用程式，角色專案範本會在 WebRole.cs 及 WorkerRole.cs 檔案中，自動為您擴充 **RoleEntryPoint** 類別。
> 
> 

## <a name="onstart-method"></a>OnStart 方法
當 Azure 將您的角色執行個體連線時，會呼叫 **OnStart** 方法。 OnStart 程式碼執行時，角色執行個體會標示為 **Busy** ，而且負載平衡器不會將任何外部流量導向至該執行個體。 您可以覆寫此方法以執行初始化工作，例如實作事件處理常式及啟動 [Azure 診斷](cloud-services-how-to-monitor.md)。

如果 **OnStart** 傳回 **true**，表示成功地初始化執行個體，而且 Azure 會呼叫 **RoleEntryPoint.Run** 方法。 如果 **OnStart** 傳回 **false**，角色會立即終止，而不執行任何計劃的關機順序。

下列程式碼範例示範如何覆寫 **OnStart** 方法。 當角色執行個體啟動，並設定將記錄資料傳輸至儲存體帳戶時，此方法會設定並啟動診斷監視器：

```csharp
public override bool OnStart()
{
    var config = DiagnosticMonitor.GetDefaultInitialConfiguration();

    config.DiagnosticInfrastructureLogs.ScheduledTransferLogLevelFilter = LogLevel.Error;
    config.DiagnosticInfrastructureLogs.ScheduledTransferPeriod = TimeSpan.FromMinutes(5);

    DiagnosticMonitor.Start("DiagnosticsConnectionString", config);

    return true;
}
```

## <a name="onstop-method"></a>OnStop 方法
在 Azure 將角色執行個體離線之後，以及處理序結束之前，會呼叫 **OnStop** 方法。 您可以覆寫此方法以呼叫讓您的角色執行個體正常關機所需的程式碼。

> [!IMPORTANT]
> 因為使用者起始關機以外的原因呼叫使用 **OnStop** 方法執行的程式碼時，其完成時間有限。 經過這個時間之後，處理序便會終止，因此您必須確定 **OnStop** 方法中的該程式碼可以快速執行，或容許完成之前不執行。 系統會在引發 **Stopping** 事件之後呼叫 **OnStop** 方法。
> 
> 

## <a name="run-method"></a>Run 方法
您可以覆寫 **Run** 方法，針對您的角色執行個體實作長時間執行的執行緒。

不需要覆寫 **Run** 方法；預設實作會啟動永久處於睡眠狀態的執行緒。 如果您覆寫 **Run** 方法，您的程式碼應該會無限期地封鎖。 如果 **Run** 方法返回，就會自動正常回收角色；換句話說，Azure 會引發 **Stopping** 事件並呼叫 **OnStop** 方法，讓您的關機順序可以在角色離線之前執行。

### <a name="implementing-the-aspnet-lifecycle-methods-for-a-web-role"></a>實作 Web 角色的 ASP.NET 生命週期方法
除了提供 **RoleEntryPoint** 類別提供的方法之外，您還可以使用 ASP.NET 生命週期方法管理 Web 角色的初始化及關機順序。 如果您要將現有的 ASP.NET 應用程式移植至 Azure，這對於相容性可能很有幫助。 系統會從 **RoleEntryPoint** 方法內呼叫 ASP.NET 生命週期方法。 系統會在 **RoleEntryPoint.OnStart** 方法完成之後，呼叫 **Application\_Start** 方法。 系統會在呼叫 **RoleEntryPoint.OnStop** 方法之前，呼叫 **Application\_End** 方法。

## <a name="next-steps"></a>後續步驟
了解如何 [建立雲端服務封裝](cloud-services-model-and-package.md)。

