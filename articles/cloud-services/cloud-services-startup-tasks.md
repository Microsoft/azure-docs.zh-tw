---
title: 在 Azure 雲端服務中執行啟動工作 | Microsoft Docs
description: 啟動工作可協助您為應用程式備妥雲端服務環境。 本文將說明啟動工作的運作方式，以及建立啟動工作的方法
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 886939be-4b5b-49cc-9a6e-2172e3c133e9
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/05/2017
ms.author: jeconnoc
ms.openlocfilehash: 6601eba90f3c3644d418ddd0a74746e1a12bcbd3
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39007774"
---
# <a name="how-to-configure-and-run-startup-tasks-for-a-cloud-service"></a>如何設定和執行雲端服務的啟動工作
您可以利用啟動工作，在角色啟動之前執行作業。 您可能想要執行的作業包括安裝元件、註冊 COM 元件、設定登錄機碼，或啟動長時間執行的處理序。

> [!NOTE]
> 啟動工作不適用於虛擬機器，只適用於雲端服務 Web 和背景工作角色。
> 
> 

## <a name="how-startup-tasks-work"></a>啟動工作的運作方式
啟動工作是在您的角色開始之前採取的動作，而且在 [ServiceDefinition.csdef] 檔案中利用 [Startup] 元素內的 [Task] 元素來定義。 啟動工作經常是批次檔，但也可以是主控台應用程式，或是啟動 PowerShell 指令碼的批次檔。

環境變數可將資訊傳入啟動工作，而本機存放區可以用來傳遞來自啟動工作的資訊。 例如，環境變數可以指定您想要安裝的程式路徑，以及可以將哪些檔案寫入本機存放區，以便日後供您的角色讀取。

啟動工作可以將資訊和錯誤記錄到 **TEMP** 環境變數所指定的目錄中。 啟動工作期間，如果在雲端上執行，**TEMP** 環境變數會解析成 C:\\Resources\\temp\\[guid].[rolename]\\RoleTemp 目錄。

啟動工作也可以在重新開機之間執行數次。 例如，每次角色回收時，都會執行啟動工作，但每次角色回收不一定會重新開機。 啟動工作的撰寫方式，應該要可讓它們順利執行多次。

啟動工作必須以零值的 **errorlevel** (或結束代碼) 做為結尾，才算完成啟動處理序。 如果啟動工作的結尾不是零值的 **errorlevel**，角色將不會啟動。

## <a name="role-startup-order"></a>角色啟動順序
以下列出 Azure 中的角色啟動程序：

1. 執行個體標示為 **啟動中** ，且沒有收到流量。
2. 所有啟動工作均會根據其 **taskType** 屬性來執行。
   
   * **simple** 的工作會以同步方式執行，一次一個。
   * **background** 和 **foreground** 工作則會以非同步方式啟動，與啟動工作平行執行。  
     
     > [!WARNING]
     > 在啟動處理序中的啟動工作階段期間，IIS 可能不會完全設定好，因此特定的角色資料可能會無法使用。 需要特定角色資料的啟動工作應該使用 [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.OnStart](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx)。
     > 
     > 
3. 角色主機處理序已啟動，且已在 IIS 中建立網站。
4. 呼叫了 [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.OnStart](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx) 方法。
5. 執行個體標示為 **就緒** ，且流量已路由傳送至執行個體。
6. 呼叫了 [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.Run](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法。

## <a name="example-of-a-startup-task"></a>啟動工作範例
啟動工作會在 [ServiceDefinition.csdef] 檔案中的 **Task** 項目定義。 **commandLine** 屬性指定啟動批次檔案或主控台命令的名稱和參數，**executionContext** 屬性指定啟動工作的權限等級，而 **taskType** 屬性則指定工作執行的方式。

在此範例中，針對啟動工作建立環境變數 **MyVersionNumber**，並設定為值 "**1.0.0.0**"。

**ServiceDefinition.csdef**：

```xml
<Startup>
    <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" >
        <Environment>
            <Variable name="MyVersionNumber" value="1.0.0.0" />
        </Environment>
    </Task>
</Startup>
```

在以下範例中， **Startup.cmd** 批次檔將 "The current version is 1.0.0.0" 一行寫入了 StartupLog.txt 檔案，此檔案位於 TEMP 環境變數所指定的目錄中。 `EXIT /B 0` 一行將確保啟動工作是以零值的 **errorlevel** 為結尾。

```cmd
ECHO The current version is %MyVersionNumber% >> "%TEMP%\StartupLog.txt" 2>&1
EXIT /B 0
```

> [!NOTE]
> 在 Visual Studio 中，啟動批次檔的 [複製到輸出目錄]屬性應該設為 [永遠複製]，才能確保將啟動批次檔正確部署至您在 Azure 上的專案 (Web 角色為 **approot\\bin**，背景工作角色為 **approot**)。
> 
> 

## <a name="description-of-task-attributes"></a>工作屬性說明
以下說明 **ServiceDefinition.csdef** 檔案中 [ServiceDefinition.csdef] 項目的屬性：

**commandLine** - 指定啟動工作的命令列：

* 此命令使用選擇性命令列參數，可開始啟動工作。
* 這通常是 .cmd 或 .bat 批次檔的檔名。
* 此工作相對於部署的 AppRoot\\Bin 資料夾。 在決定工作的路徑和檔案時，不會展開環境變數。 如果需要展開環境變數，可以建立小型 .cmd 指令碼，藉以呼叫啟動工作。
* 可以是主控台應用程式，或啟動 [PowerShell 指令碼](cloud-services-startup-tasks-common.md#create-a-powershell-startup-task)的批次檔。

**executionContext** - 指定啟動工作的權限等級。 權限等級可以是 limited (受到限制) 或 elevated (提高權限)：

* **limited**  
  啟動工作執行時會使用和角色相同的權限。 當 [Runtime] 項目的 **executionContext** 屬性也是 **limited** 時，就會用到使用者權限。
* **elevated**  
  啟動工作執行時會使用系統管理員權限。 這可讓啟動工作安裝程式、變更 IIS 組態、執行登錄變更，以及其他系統管理員層級的工作，且不會提高角色本身的權限等級。  

> [!NOTE]
> 啟動工作的權限等級不需要與角色本身相同。
> 
> 

**taskType** - 指定啟動工作執行的方式。

* **simple**  
  工作會以同步的方式執行，一次一個，並依照 [ServiceDefinition.csdef] 檔案所指定的順序。 當某個 **simple** 啟動工作結束時的 **errorlevel** 為零，就會執行下一個 **simple** 啟動工作。 如果沒有任何 **simple** 啟動工作需要執行，則會啟動角色本身。   
  
  > [!NOTE]
  > 如果 **simple** 工作結束時的 **errorlevel** 不是零，則會封鎖執行個體。 後續的 **simple** 啟動工作和角色本身將不會啟動。
  > 
  > 
  
    若要確保批次檔結束時的 **errorlevel** 為零，請在批次檔處理序結束時執行命令 `EXIT /B 0`。
* **background**  
  以非同步方式執行工作，並與角色的啟動工作平行進行。
* **foreground**  
  以非同步方式執行工作，並與角色的啟動工作平行進行。 **foreground** 和 **background** 工作之間的主要差異，在於 **foreground** 工作可避免角色在工作結束之前遭到回收或關閉。 **background** 工作則不具備這項限制功能。

## <a name="environment-variables"></a>環境變數
環境變數是將資訊傳遞給啟動工作的方式。 例如，您可以放上這些項目的路徑：含有要安裝之程式的 Blob，或角色要使用的連接埠號碼，或是各項可控制啟動工作功能的設定。

啟動工作的環境變數有兩種類型，包括靜態環境變數，還有以 [RoleEnvironment] 類別的成員為基礎的環境變數。 這兩者都位於 [ServiceDefinition.csdef] 檔案的 [Environment] 區段中，而且都使用 [Variable] 元素和 **name** 屬性。

靜態環境變數會使用 **Variable** 項目的 [Variable] 屬性。 上述範例會建立環境變數 **MyVersionNumber**，這具有靜態值 "**1.0.0.0**"。 另一個範例則是建立 **StagingOrProduction** 環境變數，您可以手動將值設為 "**staging**" 或 "**production**"，以根據 **StagingOrProduction** 環境變數的值執行不同的啟動動作。

以 RoleEnvironment 類別的成員為基礎的環境變數不會使用 **Variable** 項目的 [Variable] 屬性。 相反地，這會使用具有適當 **XPath** 屬性值的 [RoleInstanceValue] 子元素，藉此建立以 [RoleEnvironment] 類別的特定成員為基礎的環境變數。 [這裡](cloud-services-role-config-xpath.md)有存取各種 [RoleEnvironment] 值的 **XPath** 屬性值。

例如，若要建立一個環境變數，而且當執行個體在計算模擬器中執行時值為 "**true**"，而在雲端中執行時值為 "**false**"，請使用下列 [Variable] 和 [RoleInstanceValue] 元素：

```xml
<Startup>
    <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>

            <!-- Create the environment variable that informs the startup task whether it is running
                in the Compute Emulator or in the cloud. "%ComputeEmulatorRunning%"=="true" when
                running in the Compute Emulator, "%ComputeEmulatorRunning%"=="false" when running
                in the cloud. -->

            <Variable name="ComputeEmulatorRunning">
                <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
            </Variable>

        </Environment>
    </Task>
</Startup>
```

## <a name="next-steps"></a>後續步驟
了解如何透過雲端服務執行一些 [常見的啟動工作](cloud-services-startup-tasks-common.md) 。

[封裝](cloud-services-model-and-package.md) 雲端服務。  

[ServiceDefinition.csdef]: cloud-services-model-and-package.md#csdef
[Task]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Task
[Startup]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Startup
[Runtime]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Runtime
[Environment]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Environment
[Variable]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Variable
[RoleInstanceValue]: https://msdn.microsoft.com/library/azure/gg557552.aspx#RoleInstanceValue
[RoleEnvironment]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx
