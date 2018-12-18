---
title: 測試雲端服務的效能 | Microsoft Docs
description: 使用 Visual Studio 分析工具測試雲端服務的效能
services: visual-studio-online
documentationcenter: n/a
author: mikejo
manager: douge
editor: ''
ms.assetid: 7a5501aa-f92c-457c-af9b-92ea50914e24
ms.service: visual-studio-online
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 11/11/2016
ms.author: mikejo
ms.openlocfilehash: 74ffd70bdb204cee75b86a1e922ac67873a88758
ms.sourcegitcommit: 34e0b4a7427f9d2a74164a18c3063c8be967b194
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/30/2018
ms.locfileid: "30292147"
---
# <a name="testing-the-performance-of-a-cloud-service"></a>測試雲端服務的效能
## <a name="overview"></a>概觀
您可以利用下列方式測試雲端服務的效能：

* 使用 Azure 診斷來收集關於要求和連接的資訊，並檢閱可顯示從客戶觀點來看，服務執行的情況的網站統計資料。 若要開始使用，請參閱 [為 Azure 雲端服務和虛擬機器設定診斷功能](http://go.microsoft.com/fwlink/p/?LinkId=623009)。
* 使用 Visual Studio 分析工具可取得服務執行情況在計算方面的深入分析。 如本主題所述，您可以使用分析工具於服務在 Azure 中執行時來測量服務。 如需如何使用分析工具來測量服務在本機的計算模擬器中執行的效能的詳細資訊，請參閱 [使用 Visual Studio 分析工具，在計算模擬器中本機測試 Azure 雲端服務的效能](http://go.microsoft.com/fwlink/p/?LinkId=262845)。

## <a name="choosing-a-performance-testing-method"></a>選擇效能測試方法
### <a name="use-azure-diagnostics-to-collect"></a>使用 Azure 診斷以收集：
* 網頁或服務 (例如要求和連接) 的相關統計資料。
* 角色的統計資料，例如角色重新啟動的頻率。
* 記憶體使用量的整體資訊，例如記憶體回收耗費時間的百分比或執行中角色的記憶體集。

### <a name="use-the-visual-studio-profiler-to"></a>使用 Visual Studio 分析工具，以便：
* 判斷哪一個函式耗用最多時間。
* 測量每一部分密集運算的程式花費多少時間。
* 比較兩個版本服務的詳細效能報告。
* 較個別記憶體配置的層級更詳細分析記憶體配置。
* 分析多執行緒程式碼中的並行處理問題。

使用分析工具時，您可以在雲端服務於本機或在 Azure 中執行時收集資料。

### <a name="collect-profiling-data-locally-to"></a>在本機收集分析資料，以便：
* 測試部分雲端服務的效能，例如不需要實際模擬負載的特定背景工作角色的執行。
* 在受控制的情況下，隔離測試雲端服務的效能。
* 將雲端服務部署至 Azure 之前，測試雲端服務的效能。
* 私下測試雲端服務的效能，而不影響現有的部署。
* 測試服務的效能而不會產生在 Azure 中執行的費用。

### <a name="collect-profiling-data-in-azure-to"></a>在 Azure 中收集分析資料，以便：
* 在模擬或實際負載下測試雲端服務的效能。
* 使用收集分析資料的檢測方法，如稍後所述。
* 在服務執行的生產環境的相同環境中測試服務的效能。

您通常會透過模擬負載以測試在標準模式或壓力狀況下的雲端服務。

## <a name="profiling-a-cloud-service-in-azure"></a>在 Azure 中分析雲端服務
當您從 Visual Studio 發佈雲端服務時，您可以分析服務並指定可提供您想要的資訊的分析設定。 對每個角色的執行個體啟動分析工作階段。 如需有關如何從 Visual Studio 發佈服務的詳細資訊，請參閱 [從 Visual Studio 發佈至 Azure 雲端服務](https://msdn.microsoft.com/library/azure/ee460772.aspx)。

若要了解有關 Visual Studio 中的效能分析的詳細資訊，請參閱[效能分析的初學者指南](https://msdn.microsoft.com/library/azure/ms182372.aspx)和[使用分析工具來分析應用程式效能](https://msdn.microsoft.com/library/azure/z9z62c29.aspx)。

> [!NOTE]
> 發佈雲端服務時，您可以啟用 IntelliTrace 或分析。 您不能同時啟用。
> 
> 

### <a name="profiler-collection-methods"></a>分析工具收集方法
您可以對分析使用不同的收集方法，根據效能問題：

* **CPU 取樣** - 這個方法會收集對 CPU 使用率問題的初始分析很實用的應用程式統計資料。 CPU 取樣是開始大多數效能調查的建議方法。 收集 CPU 取樣資料時，對您正在分析的應用程式有很小的影響。
* **檢測** - 這個方法會收集詳細的時間資料，對重點分析以及分析輸入/輸出效能問題非常實用。 檢測方法會在執行分析期間記錄模組中函式的每個進入、結束和函式呼叫。 這個方法適合用來收集關於您的程式碼的某個區段的詳細時間資訊，以及了解輸入和輸出作業對應用程式效能的影響。 執行 32 位元作業系統的電腦會停用這個方法。 只有當您在 Azure 中 (而不是在本機計算模擬器中) 執行雲端服務時，此選項才可供使用。
* **.NET 記憶體配置** - 這個方法會使用取樣分析方法收集 .NET Framework 記憶體配置資料。 收集的資料包含配置的物件的數目和大小。
* **並行** - 這個方法會收集資源爭用資料，以及處理序與執行緒執行資料，對於分析多執行緒及多處理序的應用程式很實用。 並行方法會收集封鎖您的程式碼執行的每個事件的資料，例如當執行緒等候對應用程式資源的鎖定存取被釋放。 這個方法對於分析多執行緒應用程式很實用。
* 您也可以啟用 **階層互動分析**，提供有關一或多個資料庫通訊的多層式應用程式中的函式中同步 ADO.NET 呼叫執行時間的其他資訊。 您可以使用任何分析方法收集階層互動資料。 如需有關階層互動分析的詳細資訊，請參閱 [階層互動檢視](https://msdn.microsoft.com/library/azure/dd557764.aspx)。

## <a name="configuring-profiling-settings"></a>設定分析設定
下圖顯示如何從 [發佈 Azure 應用程式] 對話方塊中設定分析設定。

![設定分析設定](./media/vs-azure-tools-performance-profiling-cloud-services/IC526984.png)

> [!NOTE]
> 若要啟用 [啟用分析] 核取方塊，您必須要在用來發佈雲端服務的本機電腦上安裝分析工具。 根據預設，當您安裝 Visual Studio 時會一併安裝程式碼剖析工具。
> 
> 

### <a name="to-configure-profiling-settings"></a>設定分析設定
1. 在 [方案總管] 中，開啟 Azure 專案的捷徑功能表，然後選擇 [發佈] 。 如需有關如何發佈雲端服務的詳細步驟，請參閱 [使用 Azure 工具發佈雲端服務](http://go.microsoft.com/fwlink/p?LinkId=623012)。
2. 在 [發佈 Azure 應用程式] 對話方塊中，選擇 [進階設定] 索引標籤。
3. 若要啟用分析，請選取 [啟用分析]  核取方塊。
4. 若要進行分析設定，請選擇 [設定]  超連結。 [分析設定] 對話方塊隨即出現。
5. 從 [您要使用的分析方法]  選項按鈕，選擇您所需要的分析類型。
6. 若要收集層次互動分析資料，請選取 [啟用階層互動分析]  核取方塊。
7. 若要儲存設定，請選擇 [確定]  按鈕。
   
    發佈這個應用程式時，這些設定會用來建立每個角色的分析工作階段。

## <a name="viewing-profiling-reports"></a>檢視分析報告
對為雲端服務中每個角色的執行個體建立分析工作階段。 若要從 Visual Studio 檢視每個工作階段的分析報告，您可以檢視 [伺服器總管] 視窗，然後選擇 [Azure 運算] 節點以選取角色的執行個體。 您接著可以檢視分析報告，如下圖所示。

![檢視來自 Azure 的分析報告](./media/vs-azure-tools-performance-profiling-cloud-services/IC748914.png)

### <a name="to-view-profiling-reports"></a>檢視分析報告
1. 若要在 Visual Studio 中檢視 [伺服器總管] 視窗，請在功能表列上依序選擇 [檢視] 和 [伺服器總管]。
2. 選擇 [Azure 運算] 節點，然後選擇從 Visual Studio 發佈時您選取要分析的雲端服務的 Azure 部署節點。
3. 若要檢視執行個體的分析報告，請選擇服務中的角色、開啟特定執行個體的捷徑功能表，然後選擇 [檢視分析報告] 。
   
    現在便會從 Azure 下載 .vsp 檔案的報告，而下載狀態會出現在 Azure 活動記錄檔中。 下載完成時，分析報告會顯示在 Visual Studio 編輯器的索引標籤中，名為 <Role name>*<Instance Number>*<identifier>.vsp。 報告的摘要資料隨即出現。
4. 若要顯示報告的不同檢視，在 [目前檢視] 清單中，選擇您要的檢視類型。 如需詳細資訊，請參閱 [分析工具報告檢視](https://msdn.microsoft.com/library/azure/bb385755.aspx)。

## <a name="next-steps"></a>後續步驟
[偵錯雲端服務](https://msdn.microsoft.com/library/azure/ee405479.aspx)

[從 Visual Studio 發佈至 Azure 雲端服務](https://msdn.microsoft.com/library/azure/ee460772.aspx)

