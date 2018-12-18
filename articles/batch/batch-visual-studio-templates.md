---
title: 使用 Visual Studio 範本建置 Batch 解決方案 - Azure | Microsoft Docs
description: 了解 Visual Studio 專案範本如何協助您在 Azure Batch 中實作和執行計算密集型工作負載。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 5e041ae2-25af-4882-a79e-3aa63c4bfb20
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 02/27/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 5a44c249a957050afb500decd094183c71d6ca5e
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39114091"
---
# <a name="use-visual-studio-project-templates-to-jump-start-batch-solutions"></a>使用 Visual Studio 專案範本快速啟動 Batch 解決方案

Batch 的**作業管理員**和**工作處理器 Visual Studio 範本**提供了程式碼，協助您以最少的心力在 Batch 上實作並執行計算密集型工作負載。 本文件會說明這些範本，並提供其使用方式指引。

> [!IMPORTANT]
> 本文只討論適用於這兩個範本的資訊，並假設您熟悉與其相關的 Batch 服務和重要概念︰集區、計算節點、作業和工作、作業管理員工作、環境變數和其他相關資訊。 您可以在 [Azure Batch 基本概念](batch-technical-overview.md)和[適用於開發人員的 Batch 功能概觀](batch-api-basics.md)中找到詳細資訊。
> 
> 

## <a name="high-level-overview"></a>高階概述
作業管理員和工作處理器範本可用來建立兩個有用的元件︰

* 作業管理員工作，此工作會實作作業分割器，後者可將作業細分為多個可以平行且獨立執行的工作。
* 工作處理器，可用來圍繞應用程式命令列執行前置處理和後置處理。

例如，在影片轉譯案例中，作業分割器會將單一影片作業轉變成數百個或數千個會分開處理個別畫面的不同工作。 相對來說，工作處理器則會叫用為了轉譯每個畫面所需要的轉譯應用程式和所有相依處理序，並執行任何額外動作 (例如，將轉譯的畫面複製到儲存體位置)。

> [!NOTE]
> 作業管理員和工作處理器範本彼此獨立，因此您可以根據計算作業需求和個人喜好，選擇同時使用兩者或只使用其中之一。
> 
> 

如下圖所示，使用這些範本的計算作業會經歷三個階段︰

1. 用戶端程式碼 (例如，應用程式、Web 服務等) 將作業提交至 Azure 上的 Batch 服務，將其作業管理員工作指定為作業管理員程式。
2. Batch 服務在計算節點上執行作業管理員工作，作業分割器則根據作業分割器程式碼中的參數和規格，在任意數量的所需計算節點上啟動指定數量的工作處理器工作。
3. 工作處理器工作以平行方式獨立執行，處理輸入資料並產生輸出資料。

![顯示用戶端程式碼與 Batch 服務互動方式的圖表][diagram01]

## <a name="prerequisites"></a>必要條件
若要使用 Batch 範本，您需要下列項目︰

* 已安裝 Visual Studio 2015 的電腦。 目前只有針對 Visual Studio 2015 支援批次範本。
* Batch 範本，可從 [Visual Studio 資源庫][vs_gallery]取得作為 Visual Studio 擴充。 有兩種方式可取得範本︰
  
  * 在 Visual Studio 中，使用 [擴充功能和更新] 對話方塊安裝範本 (如需詳細資訊，請參閱[尋找及使用 Visual Studio 擴充][vs_find_use_ext])。 在 [擴充功能和更新]  對話方塊中，搜尋和下載下列兩個延伸模組︰
    
    * 具有作業分割器的 Azure Batch 作業管理員
    * Azure Batch 工作處理器
  * 從 Visual Studio 的線上資源庫下載範本：[Microsoft Azure Batch 專案範本][vs_gallery_templates]
* 如果您打算使用 [應用程式封裝](batch-application-packages.md) 功能將作業管理員和工作處理器部署到 Batch 計算節點，您必須連結儲存體帳戶和 Batch 帳戶。

## <a name="preparation"></a>準備工作
我們建議您建立可在其中包含作業管理員和工作處理器的方案，因為這樣便能更輕鬆地在作業管理員和工作處理器程式之間共用程式碼。 若要建立此方案，請依照下列步驟執行︰

1. 開啟 Visual Studio，然後選取 [檔案] > [新增] > [專案]。
2. 在 [範本] 底下展開 [其他專案類型]、按一下 [Visual Studio 方案]，然後選取 [空白方案]。
3. 輸入可描述應用程式和此方案用途的名稱 (例如，「LitwareBatchTaskPrograms」)。
4. 若要建立新方案，請按一下 [確定] 。

## <a name="job-manager-template"></a>作業管理員範本
作業管理員範本可協助您實作作業管理員工作以執行下列動作︰

* 將作業分割成多個工作。
* 提交這些工作以在 Batch 上執行。

> [!NOTE]
> 如需作業管理員工作的詳細資訊，請參閱 [適用於開發人員的 Batch 功能概觀](batch-api-basics.md#job-manager-task)。
> 
> 

### <a name="create-a-job-manager-using-the-template"></a>使用範本建立作業管理員
若要在稍早建立的方案中新增作業管理員，請遵循下列步驟︰

1. 在 Visual Studio 中開啟現有方案。
2. 在 [方案總管] 中，以滑鼠右鍵按一下方案，然後按一下 [新增] > [新增專案]。
3. 在 [Visual C#] 底下按一下 [雲端]，然後按一下 [具有作業分割器的 Azure Batch 作業管理員]。
4. 輸入可描述應用程式並將此專案識別為作業管理員的名稱 (例如"LitwareJobManager")。
5. 若要建立專案，按一下 [確定] 。
6. 最後，建置專案來強制 Visual Studio 載入所有參考的 NuGet 套件，以及確認專案是否有效以便能開始對其進行修改。

### <a name="job-manager-template-files-and-their-purpose"></a>作業管理員範本檔案及其用途
當您使用作業管理員範本建立專案時，它會產生三組程式碼檔案︰

* 主要程式檔 (Program.cs)。 這包含程式進入點和最上層的例外狀況處理。 通常來說，您應該不需要修改此檔案。
* 架構目錄。 其所包含的檔案負責處理作業管理員程式所進行的「重複使用」工作，像是解壓縮參數、在 Batch 作業中新增工作等。通常來說，您應該不需要修改這些檔案。
* 作業分割器檔案 (JobSplitter.cs)。 此處可供存放用於將作業分割成多個工作的應用程式特定邏輯。

當然，您可以根據作業分割邏輯的複雜度，視需要新增其他檔案來支援作業分割器程式碼。

範本也會產生標準的 .NET 專案檔案，例如 .csproj 檔案、app.config、packages.config 等等。

本節其餘部分會說明不同檔案和其程式碼結構，並解釋每個類別的用途。

![顯示作業管理員範本方案的 Visual Studio 方案總管][solution_explorer01]

**架構檔案**

* `Configuration.cs`︰封裝作業組態資料的載入，例如 Batch 帳戶詳細資料、連結的儲存體帳戶認證、作業和工作資訊，以及作業參數。 它也會透過 Configuration.EnvironmentVariable 類別提供 Batch 定義的環境變數 (請參閱 Batch 文件中適用於工作的「環境」設定) 的存取權。
* `IConfiguration.cs`︰摘要組態類別的實作，以便您可以使用假的或模擬的組態物件對作業分割器進行單元測試。
* `JobManager.cs`︰協調作業管理員程式的元件。 它負責初始化作業分割器、叫用作業分割器，以及將作業分割器傳回的工作分派給工作傳送者。
* `JobManagerException.cs`︰代表需要由作業管理員終止的錯誤。 JobManagerException 會用來包裝可在終止過程中提供特定診斷資訊的「預期」錯誤。
* `TaskSubmitter.cs`︰此類別負責將作業分割器傳回的工作新增至 Batch 作業。 JobManager 類別會將一連串工作彙總成一批，以便有效率但及時地新增到作業中，然後在每一批的背景執行緒上呼叫 TaskSubmitter.SubmitTasks。

**作業分割器**

`JobSplitter.cs`︰此類別包含用於將作業分割成多個工作的應用程式特定邏輯。 架構會叫用 JobSplitter.Split 方法以取得一連串的工作，並在方法傳回工作時將方法新增至作業中。 這是您將在其中插入作業邏輯的類別。 實作分割方法，傳回一連串代表要將作業分割成之工作的 CloudTask 執行個體。

**標準 .NET 命令列專案檔**

* `App.config`︰標準的 .NET 應用程式組態檔。
* `Packages.config`︰標準的 NuGet 套件相依性檔案。
* `Program.cs`：包含程式進入點和最上層的例外狀況處理。

### <a name="implementing-the-job-splitter"></a>實作作業分割器
當您開啟作業管理員範本專案時，專案依預設會開啟 JobSplitter.cs 檔案。 您可以如下所示使用 Split() 方法為工作負載中的工作實作分割邏輯︰

```csharp
/// <summary>
/// Gets the tasks into which to split the job. This is where you inject
/// your application-specific logic for decomposing the job into tasks.
///
/// The job manager framework invokes the Split method for you; you need
/// only to implement it, not to call it yourself. Typically, your
/// implementation should return tasks lazily, for example using a C#
/// iterator and the "yield return" statement; this allows tasks to be added
/// and to start running while splitting is still in progress.
/// </summary>
/// <returns>The tasks to be added to the job. Tasks are added automatically
/// by the job manager framework as they are returned by this method.</returns>
public IEnumerable<CloudTask> Split()
{
    // Your code for the split logic goes here.
    int startFrame = Convert.ToInt32(_parameters["StartFrame"]);
    int endFrame = Convert.ToInt32(_parameters["EndFrame"]);

    for (int i = startFrame; i <= endFrame; i++)
    {
        yield return new CloudTask("myTask" + i, "cmd /c dir");
    }
}
```

> [!NOTE]
> `Split()` 在方法中，註解區段是作業管理員範本程式碼中唯一可供您修改的區段，方法是新增用來將作業分割成不同工作的邏輯。 如果您想要修改範本的其他區段，請確定您熟悉 Batch 的運作方式，並先在幾個 [Batch 程式碼範例][github_samples]中試試看。
> 
> 

Split() 實作具有下列項目的存取權︰

* 作業參數，透過 `_parameters` 欄位。
* 代表作業的 CloudJob 物件，透過 `_job` 欄位。
* 代表作業管理員工作的 CloudJob 物件，透過 `_jobManagerTask` 欄位。

`Split()` 實作不需要直接將工作新增到作業中。 相反地，程式碼應傳回一連串的 CloudTask 物件，並由叫用作業分割器的架構類別自動新增至作業中。 通常會使用 C# 的迭代器 (`yield return`) 功能來實作作業分割器，因為這可讓工作盡快開始執行，而非等候所有要計算的工作。

**作業分割器失敗**

如果作業分割器發生錯誤，它應該︰

* 使用 C# `yield break` 陳述式終止序列，在此情況下，會將作業管理員視為成功；或者
* 擲回例外狀況，在此情況下，會將作業管理員視為失敗，並可能會根據用戶端對它的組態進行重試。

在這兩種情況下，作業分割器已傳回並新增到 Batch 作業的任何工作都有資格執行。 如果您不想讓此情況發生，則可以︰

* 終止作業，不讓它從作業分割器傳回
* 先編寫整個工作集合再將它傳回 (也就是傳回 `ICollection<CloudTask>`  `IList<CloudTask>` instead of implementing your job splitter using a C# iterat)
* 使用工作相依性讓所有工作相依於成功完成作業管理員

**作業管理員重試**

視用戶端重試設定而定，如果作業管理員失敗，Batch 服務可能會予以重試。 一般來說這很安全，因為當架構將工作新增至作業時，會忽略任何已存在的工作。 不過，如果計算工作需要很高的成本，您可能不希望因為重新計算已新增至作業的工作而產生成本，相反地，如果重新執行不保證會產生相同的工作識別碼，則「略過重複項目」的行為不會開始運作。 在這些情況下，您應該將作業分割器設計為會偵測已完成的工作而不加以重複，例如，藉由在開始產生工作之前先執行 CloudJob.ListTasks。

### <a name="exit-codes-and-exceptions-in-the-job-manager-template"></a>作業管理員範本中的結束代碼和例外狀況
結束代碼和例外狀況提供了機制來決定程式的執行結果，並可協助找出任何程式執行問題。 作業管理員範本會實作本節所述的結束代碼和例外狀況。

使用作業管理員範本實作的作業管理員工作會傳回三個可能的結束代碼︰

| 代碼 | 說明 |
| --- | --- |
| 0 |作業管理員順利完成。 作業分隔器程式碼已執行完成，而且所有工作皆已新增至作業中。 |
| 1 |作業管理員工作失敗，且程式的「預期」部分有例外狀況。 例外狀況已轉譯成 JobManagerException 與診斷資訊，如有可能，也會提供可解決失敗的建議。 |
| 2 |作業管理員工作失敗，並發生「非預期」的例外狀況。 例外狀況已記錄至標準輸出，但作業管理員無法新增任何額外的診斷或修復資訊。 |

在作業管理員工作失敗的情況下，某些工作可能仍會在錯誤發生之前就已新增至服務中。 這些工作會如常執行。 請參閱上面的「作業分割器失敗」以取得有關此程式碼路徑的討論。

例外狀況所傳回的所有資訊會寫入 stdout.txt 和 stderr.txt 檔案。 如需詳細資訊，請參閱 [錯誤處理](batch-api-basics.md#error-handling)。

### <a name="client-considerations"></a>用戶端考量
本節說明在根據此範本叫用作業管理員時的一些用戶端實作需求。 請參閱 [如何從用戶端程式碼傳遞參數和環境變數](#pass-environment-settings) ，以取得傳遞參數和環境設定的詳細資訊。

**必要認證**

若要將工作新增至 Azure Batch 作業，作業管理員工作需要您的 Azure Batch 帳戶 URL 和金鑰。 您必須在名為 YOUR_BATCH_URL 和 YOUR_BATCH_KEY 的環境變數中傳遞這些資料。 您可以在作業管理員工作的環境設定中設定這些變數。 例如，在 C# 用戶端︰

```csharp
job.JobManagerTask.EnvironmentSettings = new [] {
    new EnvironmentSetting("YOUR_BATCH_URL", "https://account.region.batch.azure.com"),
    new EnvironmentSetting("YOUR_BATCH_KEY", "{your_base64_encoded_account_key}"),
};
```
**儲存體認證**

一般而言，用戶端不需要提供連結的儲存體帳戶認證給作業管理員工作，因為 (a) 大多數作業管理員不需要明確存取連結的儲存體帳戶，而且 (b) 連結的儲存體帳戶通常會提供給所有工作，來作為作業的常見環境設定。 如果您未透過常見環境設定提供連結的儲存體帳戶，且作業管理員需要存取連結的儲存體，則應如下所示提供連結的儲存體認證︰

```csharp
job.JobManagerTask.EnvironmentSettings = new [] {
    /* other environment settings */
    new EnvironmentSetting("LINKED_STORAGE_ACCOUNT", "{storageAccountName}"),
    new EnvironmentSetting("LINKED_STORAGE_KEY", "{storageAccountKey}"),
};
```

**作業管理員工作設定**

用戶端應該將作業管理員的「killJobOnCompletion」  旗標設為 **false**。

用戶端通常可以放心地將「runExclusive」  設為 **false**。

用戶端應使用 *resourceFiles* 或 *applicationPackageReferences* 集合，將作業管理員可執行檔 (和其所需的 DLL) 部署至計算節點。

根據預設，作業管理員在失敗時不會重試。 根據作業管理員邏輯，用戶端可能會想透過 *constraints*/*maxTaskRetryCount* 啟用重試。

**作業設定**

如果作業分割器發出具有相依性的工作，用戶端必須將作業的 usesTaskDependencies 設為 true。

在作業分割器模型中，除了作業分割器所建立的工作外，用戶端通常不會想將工作新增至作業中。 因此一般來說，用戶端應該會將作業的「onAllTasksComplete」  設為 **terminatejob**。

## <a name="task-processor-template"></a>工作處理器範本
工作處理器範本可協助您實作工作處理器以執行下列動作︰

* 設定要讓每個 Batch 工作執行所需的資訊。
* 執行每個 Batch 工作所需的所有動作。
* 將工作輸出儲存至永續性儲存體。

雖然不需要工作處理器就能在 Batch 上執行工作，但使用工作處理器的主要優點是，它會提供包裝函式以在一個位置實作所有工作執行動作。 例如，如果您需要在每個工作的內容中執行數個應用程式，或者如果您需要在完成各項工作之後將資料複製到永續性儲存體。

工作處理器所執行的動作可依工作負載所需調整為任意複雜性程度和數量。 此外，藉由將所有工作動作實作到一個工作處理器中，您可以根據應用程式或工作負載需求的變更，輕易地更新或新增動作。 不過，在某些情況下，工作處理器可能不是最適合您的實作的方案，因為它會增加不必要的複雜度，例如，在執行可以從簡單命令列快速啟動的作業時。

### <a name="create-a-task-processor-using-the-template"></a>使用範本建立工作處理器
若要在稍早建立的方案中新增工作處理器，請遵循下列步驟︰

1. 在 Visual Studio 中開啟現有方案。
2. 在 [方案總管] 中，以滑鼠右鍵按一下方案、按一下 [新增]，然後按一下 [新增專案]。
3. 在 [Visual C#] 底下按一下 [雲端]，然後按一下 [Azure Batch 工作處理器]。
4. 輸入可描述應用程式並將此專案識別為工作處理器的名稱 (例如"LitwareTaskProcessor")。
5. 若要建立專案，按一下 [確定] 。
6. 最後，建置專案來強制 Visual Studio 載入所有參考的 NuGet 套件，以及確認專案是否有效以便能開始對其進行修改。

### <a name="task-processor-template-files-and-their-purpose"></a>工作處理器範本檔案及其用途
當您使用工作處理器範本建立專案時，它會產生三組程式碼檔案︰

* 主要程式檔 (Program.cs)。 這包含程式進入點和最上層的例外狀況處理。 通常來說，您應該不需要修改此檔案。
* 架構目錄。 其所包含的檔案負責處理作業管理員程式所進行的「重複使用」工作，像是解壓縮參數、在 Batch 作業中新增工作等。通常來說，您應該不需要修改這些檔案。
* 工作處理器檔案 (TaskProcessor.cs)。 此檔案可供存放用於執行工作的應用程式特定邏輯 (通常是藉由向外呼叫現有的可執行檔)。 前置和後置處理程式碼 (例如下載額外資料或上傳結果檔案) 也存放在此。

當然，您可以根據作業分割邏輯的複雜度，視需要新增其他檔案來支援工作處理器程式碼。

範本也會產生標準的 .NET 專案檔案，例如 .csproj 檔案、app.config、packages.config 等等。

本節其餘部分會說明不同檔案和其程式碼結構，並解釋每個類別的用途。

![顯示工作處理器範本方案的 Visual Studio 方案總管][solution_explorer02]

**架構檔案**

* `Configuration.cs`︰封裝作業組態資料的載入，例如 Batch 帳戶詳細資料、連結的儲存體帳戶認證、作業和工作資訊，以及作業參數。 它也會透過 Configuration.EnvironmentVariable 類別提供 Batch 定義的環境變數 (請參閱 Batch 文件中適用於工作的「環境」設定) 的存取權。
* `IConfiguration.cs`︰摘要組態類別的實作，以便您可以使用假的或模擬的組態物件對作業分割器進行單元測試。
* `TaskProcessorException.cs`︰代表需要由作業管理員終止的錯誤。 TaskProcessorException 會用來包裝可在終止過程中提供特定診斷資訊的「預期」錯誤。

**工作處理器**

* `TaskProcessor.cs`︰執行工作。 架構會叫用 TaskProcessor.Run 方法。 這是您將在其中插入工作的應用程式特定邏輯的類別。 實作 Run 方法以便︰
  * 剖析及驗證任何工作參數
  * 針對要叫用的任何外部程式撰寫命令列
  * 記錄為了偵錯所可能需要的任何診斷資訊
  * 使用該命令列開始處理序
  * 等候處理序結束
  * 擷取處理序的結束碼以判斷其已成功或失敗
  * 儲存所有想要保留在永續性儲存體的輸出檔案

**標準 .NET 命令列專案檔**

* `App.config`︰標準的 .NET 應用程式組態檔。
* `Packages.config`︰標準的 NuGet 套件相依性檔案。
* `Program.cs`：包含程式進入點和最上層的例外狀況處理。

## <a name="implementing-the-task-processor"></a>實作工作處理器
當您開啟工作處理器範本專案時，專案依預設會開啟 TaskProcessor.cs 檔案。 您可以如下所示使用 Run() 方法為工作負載中的工作實作執行邏輯︰

```csharp
/// <summary>
/// Runs the task processing logic. This is where you inject
/// your application-specific logic for decomposing the job into tasks.
///
/// The task processor framework invokes the Run method for you; you need
/// only to implement it, not to call it yourself. Typically, your
/// implementation will execute an external program (from resource files or
/// an application package), check the exit code of that program and
/// save output files to persistent storage.
/// </summary>
public async Task<int> Run()

{
    try
    {
        //Your code for the task processor goes here.
        var command = $"compare {_parameters["Frame1"]} {_parameters["Frame2"]} compare.gif";
        using (var process = Process.Start($"cmd /c {command}"))
        {
            process.WaitForExit();
            var taskOutputStorage = new TaskOutputStorage(
            _configuration.StorageAccount,
            _configuration.JobId,
            _configuration.TaskId
            );
            await taskOutputStorage.SaveAsync(
            TaskOutputKind.TaskOutput,
            @"..\stdout.txt",
            @"stdout.txt"
            );
            return process.ExitCode;
        }
    }
    catch (Exception ex)
    {
        throw new TaskProcessorException(
        $"{ex.GetType().Name} exception in run task processor: {ex.Message}",
        ex
        );
    }
}
```
> [!NOTE]
> Run() 方法中的註解區段是工作處理器範本程式碼中唯一可供您修改的區段，方法是為工作負載中的工作新增執行邏輯。 如果您想要修改範本的其他區段，請先熟悉 Batch 的運作方式，方法是檢閱 Batch 文件並在幾個 Batch 程式碼範例上進行嘗試。
> 
> 

Run() 方法負責啟動命令列、啟動一或多個處理序、等候所有處理序完成、儲存結果，以及在最後傳回結束碼。 Run() 方法可供您實作工作的處理邏輯。 工作處理器架構會為您叫用 Run() 方法；您不需要自己呼叫。

Run() 實作具有下列項目的存取權︰

* 工作參數，透過 `_parameters` 欄位。
* 作業和工作識別碼，透過 `_jobId` 和 `_taskId` 欄位。
* 工作組態，透過 `_configuration` 欄位。

**工作失敗**

萬一發生失敗，您可以擲回例外狀況以結束 Run() 方法，但這會導致最上層的例外狀況處理常式繼續控制工作結束代碼。 如果您需要控制結束代碼以便分辨不同類型的失敗，例如為了進行診斷或因為某些失敗模式應終止作業，某些則不應該，則您應該藉由傳回非零結束代碼來結束 Run() 方法。 這會成為工作結束代碼。

### <a name="exit-codes-and-exceptions-in-the-task-processor-template"></a>工作處理器範本中的結束代碼和例外狀況
結束代碼和例外狀況提供了機制來決定程式的執行結果，並可協助找出任何程式執行問題。 工作處理器範本會實作本節所述的結束代碼和例外狀況。

使用工作處理器範本實作的工作處理器工作會傳回三個可能的結束代碼︰

| 代碼 | 說明 |
| --- | --- |
| [Process.ExitCode][process_exitcode] |工作處理器已執行完成。 請注意，這並非表示您叫用的程式已成功，只表示工作處理器已成功叫用程式並執行任何後置處理，而沒有例外狀況。 結束代碼的意義取決於所叫用的程式，一般來說，結束代碼 0 表示程式已成功，任何其他結束代碼則表示程式失敗。 |
| 1 |工作處理器失敗，且程式的「預期」部分有例外狀況。 例外狀況已轉譯成 `TaskProcessorException` 與診斷資訊，如有可能，也會提供可解決失敗的建議。 |
| 2 |工作處理器失敗，並發生「非預期」的例外狀況。 例外狀況已記錄至標準輸出，但工作處理器無法新增任何額外的診斷或修復資訊。 |

> [!NOTE]
> 如果您叫用的程式使用結束代碼 1 和 2 來指出特定失敗模式，則使用結束代碼 1 和 2 來代表工作處理器錯誤將造成模稜兩可的狀況。 您可以編輯 Program.cs 檔案中的例外狀況案例，將這些工作處理器錯誤碼變更為可資區分的結束代碼。
> 
> 

例外狀況所傳回的所有資訊會寫入 stdout.txt 和 stderr.txt 檔案。 如需詳細資訊，請參閱 Batch 文件中的＜錯誤處理＞。

### <a name="client-considerations"></a>用戶端考量
**儲存體認證**

如果工作處理器會使用 Azure Blob 儲存體來保存輸出 (例如使用檔案慣例協助程式程式庫)，則它需要存取雲端儲存體帳戶認證或包含共用存取簽章 (SAS) 的 Blob 容器 URL。 範本支援透過常見環境變數來提供認證。 用戶端可以如下所示傳遞儲存體認證︰

```csharp
job.CommonEnvironmentSettings = new [] {
    new EnvironmentSetting("LINKED_STORAGE_ACCOUNT", "{storageAccountName}"),
    new EnvironmentSetting("LINKED_STORAGE_KEY", "{storageAccountKey}"),
};
```

然後，就可以透過 `_configuration.StorageAccount` 屬性在 TaskProcessor 類別中使用儲存體帳戶。

如果您想要使用具有 SAS 的容器 URL，您也可以透過作業的常見環境設定傳遞此 URL，但工作處理器範本目前未內建支援此 URL。

**儲存體設定**

建議用戶端或作業管理員工作先建立工作所需的任何容器，再將工作新增至作業。 如果您使用具有 SAS 的容器 URL 就必須這麼做，因為這樣的 URL 並未包含建立容器的權限。 即使您傳遞的是儲存體帳戶認證仍建議您這麼做，因為它會儲存每一項必須在容器上呼叫 CloudBlobContainer.CreateIfNotExistsAsync 的工作。

## <a name="pass-parameters-and-environment-variables"></a>傳遞參數和環境變數
### <a name="pass-environment-settings"></a>傳遞環境設定
用戶端可以環境設定的形式將資訊傳遞給作業管理員工作。 接著，作業管理員工作可在產生做為計算作業一部分來執行的工作處理器工作時使用這項資訊。 可以環境設定形式傳遞的資訊範例如下︰

* 儲存體帳戶名稱和帳戶金鑰
* Batch 帳戶 URL
* Batch 帳戶金鑰

Batch 服務具有簡單的機制，可在 [Microsoft.Azure.Batch.JobManagerTask][net_jobmanagertask] 中使用 `EnvironmentSettings` 屬性，將環境設定傳遞至作業管理員工作。

例如，若要取得 Batch 帳戶的 `BatchClient` 執行個體，您可以環境變數的形式從用戶端程式碼傳遞 Batch 帳戶的 URL 和共用金鑰認證。 同樣地，若要存取連結至 Batch 帳戶的儲存體帳戶，您可使用環境變數的形式傳遞儲存體帳戶名稱和儲存體帳戶金鑰。

### <a name="pass-parameters-to-the-job-manager-template"></a>將參數傳遞至作業管理員範本
在許多情況下，最好將每個作業的參數傳遞至作業管理員工作，以便控制作業分割處理序或是設定作業的工作。 若要這麼做，可以將名為 parameters.json 的 JSON 檔案上傳做為作業管理員工作的資源檔。 然後，參數就可以在作業管理員範本的 `JobSplitter._parameters` 欄位中變為可供使用。

> [!NOTE]
> 內建的參數處理常式只支援字串對字串的字典。 如果您想要以參數值的形式傳遞複雜 JSON 值，就必須以字串形式傳遞並在作業分割器中進行剖析，或是修改架構的 `Configuration.GetJobParameters` 方法。
> 
> 

### <a name="pass-parameters-to-the-task-processor-template"></a>將參數傳遞給工作處理器範本
您也可以使用工作處理器範本將參數傳遞至所實作的個別工作。 就像使用作業管理員範本一樣，工作處理器範本會尋找名為

parameters.json 的資源檔案，如果找到，即會將它載入以做為參數字典。 有幾個選項可用來將參數傳遞給工作處理器工作︰

* 重複使用作業參數 JSON。 這適用於唯一的參數都是全作業參數時 (例如轉譯高度和寬度)。 若要這樣做，請於在作業分割器中建立 CloudTask 時，從作業管理員工作的 ResourceFiles (`JobSplitter._jobManagerTask.ResourceFiles`) 將 parameters.json 資源檔物件的參考新增至 CloudTask 的 ResourceFiles 集合。
* 產生和上傳工作特定的 parameters.json 文件做為作業分隔器執行的一部分，並在工作的資源檔集合中參考該 Blob。 如果不同的工作有不同的參數，就必須這麼做。 以參數形式將畫面索引傳遞至工作的 3D 轉譯案例便是可能的範例。

> [!NOTE]
> 內建的參數處理常式只支援字串對字串的字典。 如果您想要以參數值的形式傳遞複雜 JSON 值，就必須以字串形式傳遞並在工作處理器中進行剖析，或是修改架構的 `Configuration.GetTaskParameters` 方法。
> 
> 

## <a name="next-steps"></a>後續步驟
### <a name="persist-job-and-task-output-to-azure-storage"></a>將作業和工作輸出保存到 Azure 儲存體
開發 Batch 解決方案時的另一個實用工具是 [Azure Batch 檔案慣例][nuget_package]。 在 Batch .NET 應用程式中使用此 .NET 類別庫 (目前為預覽版) 可在 Azure 儲存體中輕鬆地儲存或擷取工作輸出。 [保存 Azure Batch 作業和工作輸出](batch-task-output.md) 包含類別庫及其使用方式的完整討論。


[net_jobmanagertask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobmanagertask.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[process_exitcode]: https://msdn.microsoft.com/library/system.diagnostics.process.exitcode.aspx
[vs_gallery]: https://visualstudiogallery.msdn.microsoft.com/
[vs_gallery_templates]: https://go.microsoft.com/fwlink/?linkid=820714
[vs_find_use_ext]: https://msdn.microsoft.com/library/dd293638.aspx

[diagram01]: ./media/batch-visual-studio-templates/diagram01.png
[solution_explorer01]: ./media/batch-visual-studio-templates/solution_explorer01.png
[solution_explorer02]: ./media/batch-visual-studio-templates/solution_explorer02.png
