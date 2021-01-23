---
title: 預設 TEMP 資料夾對角色而言太小 | Microsoft Docs
description: 雲端服務角色的 TEMP 資料夾空間量有限。 本文提供關於如何避免用盡空間的一些建議。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 1b7bfb47168c31f9e2e1b7e40764439118c00805
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98743197"
---
# <a name="default-temp-folder-size-is-too-small-on-a-cloud-service-classic-webworker-role"></a>雲端服務 (傳統) web/背景工作角色的預設 TEMP 資料夾大小太小

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

雲端服務背景工作角色或 Web 角色的預設暫存目錄大小上限為 100 MB，可能會在某個時間點達到。 本文說明如何避免用盡暫存目錄的空間。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="why-do-i-run-out-of-space"></a>為何會用盡空間？
您應用程式中執行的程式碼可以使用標準 Windows 環境變數 TEMP 和 TMP。 TEMP 和 TMP 都會指向大小上限為 100 MB 的單一目錄。 任何儲存在此目錄中的資料都不會跨雲端服務的生命週期而保存；如果雲端服務中的角色執行個體被回收，就會清除目錄。

## <a name="suggestion-to-fix-the-problem"></a>修正此問題的建議
實作下列其中一個替代方案：

* 設定本機儲存資源並直接加以存取，而不要使用 TEMP 或 TMP。 若要從您應用程式內執行的程式碼存取本機儲存資源，請呼叫 [RoleEnvironment.GetLocalResource](/previous-versions/azure/reference/ee772845(v=azure.100)) 方法。
* 設定本機儲存資源，並將 TEMP 和 TMP 目錄指向本機儲存資源的路徑。 此修改應在 [RoleEntryPoint.OnStart](/previous-versions/azure/reference/ee772851(v=azure.100)) 方法內執行。

下列程式碼範例說明如何在 OnStart 方法內修改 TEMP 和 TMP 的目標目錄：

```csharp
using System;
using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRole1
{
    public class WorkerRole : RoleEntryPoint
    {
        public override bool OnStart()
        {
            // The local resource declaration must have been added to the
            // service definition file for the role named WorkerRole1:
            //
            // <LocalResources>
            //    <LocalStorage name="CustomTempLocalStore"
            //                  cleanOnRoleRecycle="false"
            //                  sizeInMB="1024" />
            // </LocalResources>

            string customTempLocalResourcePath =
            RoleEnvironment.GetLocalResource("CustomTempLocalStore").RootPath;
            Environment.SetEnvironmentVariable("TMP", customTempLocalResourcePath);
            Environment.SetEnvironmentVariable("TEMP", customTempLocalResourcePath);

            // The rest of your startup code goes here…

            return base.OnStart();
        }
    }
}
```

## <a name="next-steps"></a>後續步驟
請參閱說明 [如何增加 Azure Web 角色 ASP.NET 暫存資料夾大小](/archive/blogs/kwill/how-to-increase-the-size-of-the-windows-azure-web-role-asp-net-temporary-folder)的部落格。

檢視更多雲端服務的 [疑難排解文章](/visualstudio/azure/vs-azure-tools-debugging-cloud-services-overview) 。

若要了解如何利用 Azure PaaS 電腦診斷資料，對雲端服務角色的問題進行疑難排解，請檢視 [Kevin Williamson 的部落格系列](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data)。