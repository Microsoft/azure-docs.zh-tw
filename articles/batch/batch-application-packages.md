---
title: 在運算節點上安裝應用程式套件 - Azure Batch | Microsoft Docs
description: 使用 Azure Batch 的應用程式套件功能輕鬆地管理多個應用程式和版本，以便安裝在 Batch 計算節點。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 3b6044b7-5f65-4a27-9d43-71e1863d16cf
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 06/15/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 333161042e968b4baf4b962869d688fd0b696b24
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094130"
---
# <a name="deploy-applications-to-compute-nodes-with-batch-application-packages"></a>使用 Batch 應用程式套件將應用程式部署至計算節點

Azure Batch 的應用程式套件功能可讓您輕鬆管理工作應用程式並將其部署到集區中的計算節點。 透過應用程式套件，您可以上傳和管理工作所執行的多個應用程式版本，包括其支援檔案。 接著，您可以將一或多個這種類型的應用程式自動部署到集區中的計算節點。

在此文章中，您會了解如何使用 Azure 入口網站上傳和管理應用程式套件。 然後，您會了解如何使用 [Batch .NET][api_net] 程式庫將套件安裝在集區的計算節點上。

> [!NOTE]
> 
> 在 2017 年 7 月 5 日之後建立的所有 Batch 集區都支援應用程式套件。 只有在使用雲端服務設定建立集區時，在 2016 年 3 月 10 日與 2017 年 7 月 5 日之間所建立的 Batch 集區上才支援應用程式套件。 在 2016 年 3 月 10 日之前建立的 Batch 集區不支援應用程式套件。
>
> 用來建立和管理應用程式套件的 API 屬於 [Batch Management .NET][api_net_mgmt] 程式庫的一部分。 用來在計算節點上安裝應用程式套件的 API 則屬於 [Batch .NET][api_net] 程式庫的一部分。 其他語言的類似功能在其可用 Batch API 中。 
>
> 此處所述的應用程式套件功能取代了舊版服務中的「Batch Apps」功能。
> 
> 

## <a name="application-package-requirements"></a>應用程式套件需求
若要使用應用程式套件，您必須[將 Azure 儲存體帳戶連結](#link-a-storage-account)到 Batch 帳戶。

## <a name="about-applications-and-application-packages"></a>關於應用程式和應用程式套件
在 Azure Batch 中，「應用程式」  是指一組已建立版本的二進位檔，這些檔案可自動下載到集區中的計算節點。 「應用程式套件」指的是這些二進位檔的「特定組合」，其代表應用程式的特定「版本」。

![應用程式和應用程式套件的高階圖表][1]

### <a name="applications"></a>[應用程式]
Batch 中的應用程式包含一或多個應用程式套件，並且會指定應用程式的組態選項。 例如，應用程式可以指定要安裝在計算節點上的預設應用程式套件版本，以及應用程式的套件是否可以更新或刪除。

### <a name="application-packages"></a>應用程式套件
應用程式套件為 .zip 檔案，其中包含工作執行應用程式所需的應用程式二進位檔和支援檔案。 每個應用程式套件都代表特定版本的應用程式。

您可以在集區和工作層級指定應用程式套件。 當您建立集區或工作時，您可以指定這些套件之中的一個或多個，以及 (選擇性) 指定版本。

* **集區應用程式套件**會部署到集區中的「每個」節點。 當節點加入集區以及重新啟動或重新安裝映像時，就會部署應用程式。
  
    當集區中的所有節點都執行某作業的工作時，便適合使用集區應用程式套件。 您可以在建立集區時指定一或多個應用程式套件，而且可以新增或更新現有集區的套件。 如果您更新現有集區的應用程式套件，您必須重新啟動它的節點，以安裝新的套件。
* **工作應用程式套件** 只會部署到排程要執行工作的計算節點。 如果節點上已有指定的應用程式套件和版本，則不會重新部署，而會使用現有套件。
  
    在共用集區的環境中，工作應用程式套件裝很有用：不同的作業會在一個集區上執行，而某個作業完成時並不會刪除該集區。 如果您的作業擁有的工作少於集區中的節點，工作應用程式套件可以減少資料傳輸，因為您的應用程式只會部署至執行工作的節點。
  
    其他可受益於工作應用程式套件的情節為執行大型應用程式，但只用於少數工作的作業。 例如，前置處理或合併應用程式非常龐大的前置處理階段或合併工作，將會受益於使用工作應用程式套件。

> [!IMPORTANT]
> Batch 帳戶中的應用程式和應用程式套件數目，以及應用程式套件的大小上限有其限制。 如需這些限制的詳細資料，請參閱 [Azure Batch 服務的配額和限制](batch-quota-limit.md) 。
> 
> 

### <a name="benefits-of-application-packages"></a>應用程式套件的優點
應用程式套件可以簡化 Batch 解決方案中的程式碼，以及降低工作所執行之應用程式的必要管理成本。

使用應用程式套件，集區的啟動工作就不需要指定在節點上安裝一長串的個別資源檔案。 您不需要在 Azure 儲存體中或在節點上手動管理應用程式檔案的多個版本。 再者，您也不必費心產生 [SAS URL](../storage/common/storage-dotnet-shared-access-signature-part-1.md) 來提供這些檔案在儲存體帳戶中的存取權限。 Batch 會在背景中與 Azure 儲存體合作來儲存應用程式套件，並將其部署到計算節點。

> [!NOTE] 
> 啟動工作的總大小必須小於或等於 32768 個字元，包括資源檔案和環境變數。 如果啟動工作超過此限制，就可另外選擇使用應用程式套件。 您可以也建立包含資源檔的壓縮封存、將它作為 blob 上傳到 Azure 儲存體，然後從啟動工作的命令列將它解壓縮。 
>
>

## <a name="upload-and-manage-applications"></a>上傳及管理應用程式
您可以使用 [Azure 入口網站][portal]或 Batch Management API 來管理 Batch 帳戶中的應用程式套件。 在接下來的幾節中，我們會先示範如何連結儲存體帳戶，接著討論如何使用入口網站來新增應用程式和套件以及管理它們。

### <a name="link-a-storage-account"></a>連結儲存體帳戶
若要使用應用程式套件，您必須先將 [Azure 儲存體帳戶](batch-api-basics.md#azure-storage-account)連結到 Batch 帳戶。 如果您尚未設定儲存體帳戶，Azure 入口網站會在您第一次按下 Batch 帳戶中的 [應用程式] 時顯示警告。



![Azure 入口網站中的「未設定儲存體帳戶」警告][9]

Batch 服務會使用相關聯的儲存體帳戶來儲存應用程式套件。 在連結兩個帳戶之後，Batch 便能將儲存在連結之儲存體帳戶中的套件自動部署到計算節點。 若要將儲存體帳戶連結至 Batch 帳戶，請按一下 [警告] 視窗上的 [儲存體帳戶]，然後再次按一下 [儲存體帳戶]。

![Azure 入口網站中的選擇儲存體帳戶刀鋒視窗][10]

我們建議您建立「專門」用來與 Batch 帳戶搭配使用的儲存體帳戶，並在此處選取它。 在建立儲存體帳戶之後，您可以使用 [儲存體帳戶]  視窗，將它連結到 Batch 帳戶。

> [!NOTE] 
> 您目前無法使用應用程式套件搭配使用已設定[防火牆規則](../storage/common/storage-network-security.md)的 Azure 儲存體帳戶。
> 

Batch 服務會使用 Azure 儲存體將應用程式套件儲存為區塊 Blob。 針對區塊 Blob 資料，您需[支付標準費用][storage_pricing]，而每個套件的大小則不能超過[區塊 Blob 大小上限](../storage/common/storage-scalability-targets.md#azure-blob-storage-scale-targets)。 請務必考量應用程式套件的大小和數目，並定期移除過時的套件以降低成本。
> 
> 

### <a name="view-current-applications"></a>檢視目前的應用程式
若要檢視 Batch 帳戶中的應用程式，請在檢視 [Batch 帳戶] 視窗時按一下左側功能表中的 [應用程式] 功能表項目。

![應用程式圖格][2]

選取此功能表選項會開啟 [應用程式] 視窗：

![列出應用程式][3]

此視窗會顯示帳戶中每個應用程式的識別碼，以及下列屬性︰

* **套件**：與此應用程式相關聯的版本號碼。
* **預設版本**：如果您在指定集區的應用程式時未指定版本，系統會安裝的應用程式版本。 此設定是選擇性的。
* **允許更新**：此值會指定是否允許更新、刪除和新增套件。 如果此值設為 [否]，應用程式會停用套件的更新和刪除， 只能新增新的應用程式套件版本。 預設值為 [是]。

### <a name="view-application-details"></a>檢視應用程式詳細資料
若要查看應用程式的詳細資料，請在 [應用程式] 視窗中選取應用程式。

![應用程式詳細資料][4]

在應用程式詳細資料中，您可以配置應用程式的以下設定。

* **允許更新**：指定是否可更新或刪除應用程式的應用程式套件。 請參閱此文章稍後的＜更新或刪除應用程式套件＞。
* **預設版本**：指定要部署至計算節點的預設應用程式套件。
* **顯示名稱**：指定 Batch 解決方案在顯示應用程式相關資訊時 (例如，在透過 Batch 提供給客戶之服務的 UI 中)，可以使用的「易記」名稱。

### <a name="add-a-new-application"></a>加入新的應用程式
若要建立新應用程式，請新增應用程式套件並指定新的唯一應用程式識別碼。 使用新應用程式識別碼新增的第一個應用程式套件也會建立新的應用程式。

按一下 [應用程式] > [新增]。

![Azure 入口網站中的新增應用程式刀鋒視窗][5]

[新增應用程式]  視窗提供以下欄位，供您指定新應用程式和應用程式套件的設定。

**應用程式識別碼**

此欄位能指定新應用程式的識別碼，其須符合標準 Azure Batch 識別碼驗證規則的規範。 提供應用程式識別碼的規則如下所示：

* 在 Windows 節點上，識別碼可包含英數字元、連字號及底線的任意組合。 在 Linux 節點上，只允許使用英數字元和底線。
* 不能包含超過 64 個字元。
* 在 Batch 帳戶內必須是唯一的。
* 需保留大小寫和區分大小寫。

**版本**

此欄位會指定要上傳的應用程式套件版本。 版本字串須符合以下驗證規則的規範︰

* 在 Windows 節點上，版本字串可包含英數字元、連字號、底線及句號的任意組合。 在 Linux 節點上，版本字串可以只包含英數字元和底線。
* 不能包含超過 64 個字元。
* 在應用程式內必須是唯一的。
* 需保留大小寫和區分大小寫。

**應用程式套件**

此欄位指定包含應用程式執行所需之應用程式二進位檔和支援檔案的 .zip 檔案。 按一下 [選取檔案]  方塊或資料夾圖示，以瀏覽及選取包含應用程式檔案的 .zip 檔案。

在選取檔案之後，請按一下 [確定]  以開始上傳到 Azure 儲存體。 上傳作業完成時，入口網站會顯示通知。 因上傳之檔案的大小和網路連線速度不盡相同，此作業可能需要花費一些時間。

> [!WARNING]
> 上傳作業完成之前，請勿關閉 [新增應用程式] 視窗。 否則上傳程序會停止。
> 
> 

### <a name="add-a-new-application-package"></a>加入新應用程式套件
若要為現有的應用程式新增應用程式套件版本，請在 [應用程式] 視窗中選取應用程式，然後按一下 [套件] > [新增]。

![Azure 入口網站中的新增應用程式套件刀鋒視窗][8]

如您所見，欄位與 [新增應用程式] 視窗中的欄位相符，但 [應用程式識別碼] 方塊已停用。 依照和新增應用程式相同的方式，指定新套件的 [版本]，瀏覽至您的**應用程式套件** .zip 檔案，然後按一下 [確定] 以上傳套件。

### <a name="update-or-delete-an-application-package"></a>更新或刪除應用程式套件
若要更新或刪除現有的應用程式套件，請開啟應用程式的詳細資料，按一下 [套件]，針對要修改的應用程式套件資料列，按一下**省略符號**，然後選取要執行的動作。

![在 Azure 入口網站中更新或刪除套件][7]

更新

當您按一下 [更新] 時，[更新套件] 視窗隨即出現。 此視窗與 [新增應用程式套件]  視窗相似，只不過已啟用套件選取欄位，因此您可以指定要上傳的新 ZIP 檔案。

![Azure 入口網站中的更新套件刀鋒視窗][11]

**刪除**

當您按一下 [刪除] 時，系統會要求您確認要刪除套件版本，隨後 Batch 會從 Azure 儲存體中刪除該套件。 如果您刪除應用程式的預設版本，系統會移除應用程式的 [預設版本]  設定。

![刪除應用程式][12]

## <a name="install-applications-on-compute-nodes"></a>將應用程式安裝在計算節點上
您已了解如何使用 Azure 入口網站管理應用程式套件，接下來我們可以討論如何使用 Batch 工作，將它們部署到計算節點並執行它們。

### <a name="install-pool-application-packages"></a>安裝集區應用程式套件
若要將應用程式套件安裝在集區中的所有計算節點上，請為集區指定一或多個應用程式套件「參考」  。 您針對集區指定的應用程式套件會在每個電腦節點加入集區時，以及該節點重新啟動或重新安裝映像時，安裝於該節點上。

在 Batch .NET 中，請在建立新集區時指定一或多個 [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref]，或為現有集區指定。 [ApplicationPackageReference][net_pkgref] 類別能指定要安裝在集區之計算節點上的應用程式識別碼和版本。

```csharp
// Create the unbound CloudPool
CloudPool myCloudPool =
    batchClient.PoolOperations.CreatePool(
        poolId: "myPool",
        targetDedicatedComputeNodes: 1,
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Specify the application and version to install on the compute nodes
myCloudPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "litware",
        Version = "1.1001.2b" }
};

// Commit the pool so that it's created in the Batch service. As the nodes join
// the pool, the specified application package is installed on each.
await myCloudPool.CommitAsync();
```

> [!IMPORTANT]
> 如果應用程式套件部署基於任何因素而失敗，Batch 服務會將節點標示為[無法使用][net_nodestate]，而且不會在該節點上排程任何要執行的工作。 在此情況下，您應該 **重新啟動** 節點以重新初始化套件部署。 重新啟動節點也會在節點上再次啟用工作排程。
> 
> 

### <a name="install-task-application-packages"></a>安裝工作應用程式套件
類似於集區，您可以為工作指定應用程式套件「參考」  。 在節點上排程要執行的工作時，會先下載並解壓縮套件，再執行工作的命令列。 如果節點上已安裝指定的套件和版本，則不會下載套件，而會使用現有套件裝。

若要安裝工作應用程式套件，請設定工作的 [CloudTask][net_cloudtask].[ApplicationPackageReferences][net_cloudtask_pkgref] 屬性︰

```csharp
CloudTask task =
    new CloudTask(
        "litwaretask001",
        "cmd /c %AZ_BATCH_APP_PACKAGE_LITWARE%\\litware.exe -args -here");

task.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference
    {
        ApplicationId = "litware",
        Version = "1.1001.2b"
    }
};
```

## <a name="execute-the-installed-applications"></a>執行安裝的應用程式
您為集區或工作所指定的套件會下載並解壓縮至節點的 `AZ_BATCH_ROOT_DIR` 中的具名目錄。 Batch 也會建立包含具名目錄路徑的環境變數。 在參考節點上的應用程式時，工作的命令列會使用這個環境變數。 

在 Windows 節點上，變數格式如下：

```
Windows:
AZ_BATCH_APP_PACKAGE_APPLICATIONID#version
```

在 Linux 節點上，格式稍有不同。 句號 (.)、連字號 (-) 和數字記號 (#) 已壓平合併為環境變數中的底線。 另外請注意，會保留應用程式識別碼的大小寫。 例如︰

```
Linux:
AZ_BATCH_APP_PACKAGE_applicationid_version
```

`APPLICATIONID` 和 `version` 是對應於為部署所指定的應用程式和套件版本的值。 例如，如果您指定應該在 Windows 節點上安裝 2.7 版的「Blender」應用程式，您的工作命令列就會使用此環境變數來存取其檔案︰

```
Windows:
AZ_BATCH_APP_PACKAGE_BLENDER#2.7
```

在 Linux 節點上，以此格式指定環境變數。 將句號 (.)、連字號 (-) 和數字符號 (#) 壓平合併化為底線，並保留應用程式識別碼的大小寫：

```
Linux:
AZ_BATCH_APP_PACKAGE_blender_2_7
``` 

當您上傳應用程式套件時，您可以指定要部署至運算節點的預設版本。 如果您已經指定應用程式的預設版本，當您參考應用程式時就可以省略版本尾碼。 您可以在 Azure 入口網站中的 [應用程式] 視窗上指定預設應用程式版本，如[上傳及管理應用程式](#upload-and-manage-applications)中所示。

例如，如果您設定「2.7」作為「Blender」應用程式的預設版本，且您的工作是參考下列環境變數，您的 Windows 節點就會執行 2.7 版：

`AZ_BATCH_APP_PACKAGE_BLENDER`

下列程式碼片段會顯示工作命令列範例，其可啟動「Blender」  應用程式的預設版本︰

```csharp
string taskId = "blendertask01";
string commandLine =
    @"cmd /c %AZ_BATCH_APP_PACKAGE_BLENDER%\blender.exe -args -here";
CloudTask blenderTask = new CloudTask(taskId, commandLine);
```

> [!TIP]
> 如需計算節點環境設定的詳細資訊，請參閱 [Batch 功能概觀](batch-api-basics.md)中的[工作的環境設定](batch-api-basics.md#environment-settings-for-tasks)。
> 
> 

## <a name="update-a-pools-application-packages"></a>更新集區的應用程式套件
如果您已設定現有集區的應用程式套件，可以指定集區的新套件。 如果您為集區指定新的套件參考，則會適用下列情況：

* Batch 服務會在加入新增集區的所有新節點，以及任何重新啟動或重新安裝映像的現有節點上安裝新指定的套件。
* 在更新套件參考時已存在集區中的計算節點不會自動安裝新應用程式套件。 這些計算節點必須重新啟動或重新安裝映像才能接收新套件。
* 部署新的套件之後，所建立的環境變數會反映新的應用程式套件參考。

在此範例中，現有集區已將 Blender 應用程式的 2.7 版設定為其中一個 [CloudPool][net_cloudpool].[ApplicationPackageReferences][net_cloudpool_pkgref]。 若要將集區的節點更新為 2.76b 版，請將 [ApplicationPackageReference][net_pkgref] 指定為新版本並認可變更。

```csharp
string newVersion = "2.76b";
CloudPool boundPool = await batchClient.PoolOperations.GetPoolAsync("myPool");
boundPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "blender",
        Version = newVersion }
};
await boundPool.CommitAsync();
```

既然您已設定新版本，Batch 服務就會將 2.76b 版安裝到任何加入集區的「新」節點。 若要將 2.76b 安裝在「已存在」  集區中的節點上，請將節點重新啟動或重新安裝映像。 請注意，重新啟動的節點會保留前次套件部署的檔案。

## <a name="list-the-applications-in-a-batch-account"></a>列出 Batch 帳戶中的應用程式
您可以使用 [ApplicationOperations][net_appops].[ListApplicationSummaries][net_appops_listappsummaries] 方法列出 Batch 帳戶中的應用程式和應用程式套件。

```csharp
// List the applications and their application packages in the Batch account.
List<ApplicationSummary> applications = await batchClient.ApplicationOperations.ListApplicationSummaries().ToListAsync();
foreach (ApplicationSummary app in applications)
{
    Console.WriteLine("ID: {0} | Display Name: {1}", app.Id, app.DisplayName);

    foreach (string version in app.Versions)
    {
        Console.WriteLine("  {0}", version);
    }
}
```

## <a name="wrap-up"></a>總結
透過應用程式套件，您可以協助客戶選取其作業適用的應用程式，以及指定在以啟用 Batch 功能的服務處理作業時所要使用的確切版本。 您也可以在服務中提供讓客戶上傳及追蹤其應用程式的功能。

## <a name="next-steps"></a>後續步驟
* [Batch REST API][api_rest] 也提供應用程式套件的使用支援。 例如，請參閱[將集區新增至帳戶][rest_add_pool]中的 [applicationPackageReferences][rest_add_pool_with_packages] 項目，以取得如何使用 REST API 來指定要安裝之套件的相關資訊。 如需如何使用 Batch REST API 來取得應用程式資訊的詳細資料，請參閱[應用程式][rest_applications]。
* 了解如何以程式設計方式 [使用 Batch Management .NET 管理 Azure Batch 帳戶和配額](batch-management-dotnet.md)。 [Batch Management .NET][api_net_mgmt] 程式庫可以啟用 Batch 應用程式或服務的帳戶建立和刪除功能。

[api_net]: https://docs.microsoft.com/dotnet/api/overview/azure/batch/client?view=azure-dotnet
[api_net_mgmt]: https://docs.microsoft.com/dotnet/api/overview/azure/batch/management?view=azure-dotnet
[api_rest]: https://docs.microsoft.com/rest/api/batchservice/
[batch_mgmt_nuget]: https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/
[github_samples]: https://github.com/Azure/azure-batch-samples
[storage_pricing]: https://azure.microsoft.com/pricing/details/storage/
[net_appops]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationoperations.aspx
[net_appops_listappsummaries]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationoperations.listapplicationsummaries.aspx
[net_cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_cloudpool_pkgref]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.applicationpackagereferences.aspx
[net_cloudtask]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudtask.aspx
[net_cloudtask_pkgref]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudtask.applicationpackagereferences.aspx
[net_nodestate]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.state.aspx
[net_pkgref]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.applicationpackagereference.aspx
[portal]: https://portal.azure.com
[rest_applications]: https://msdn.microsoft.com/library/azure/mt643945.aspx
[rest_add_pool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[rest_add_pool_with_packages]: https://msdn.microsoft.com/library/azure/dn820174.aspx#bk_apkgreference

[1]: ./media/batch-application-packages/app_pkg_01.png "應用程式套件高階圖表"
[2]: ./media/batch-application-packages/app_pkg_02.png "Azure 入口網站中的應用程式圖格"
[3]: ./media/batch-application-packages/app_pkg_03.png "Azure 入口網站中的應用程式刀鋒視窗"
[4]: ./media/batch-application-packages/app_pkg_04.png "Azure 入口網站中的應用程式詳細資料刀鋒視窗"
[5]: ./media/batch-application-packages/app_pkg_05.png "Azure 入口網站中的新增應用程式刀鋒視窗"
[7]: ./media/batch-application-packages/app_pkg_07.png "Azure 入口網站中的更新或刪除套件下拉式清單"
[8]: ./media/batch-application-packages/app_pkg_08.png "Azure 入口網站中的新增應用程式套件刀鋒視窗"
[9]: ./media/batch-application-packages/app_pkg_09.png "沒有連結的儲存體帳戶警示"
[10]: ./media/batch-application-packages/app_pkg_10.png "Azure 入口網站中的選擇儲存體帳戶刀鋒視窗"
[11]: ./media/batch-application-packages/app_pkg_11.png "Azure 入口網站中的更新套件刀鋒視窗"
[12]: ./media/batch-application-packages/app_pkg_12.png "Azure 入口網站中的刪除套件確認對話方塊"
