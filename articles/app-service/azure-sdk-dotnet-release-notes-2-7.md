---
title: Azure SDK for .NET 2.7 和 .NET 2.7.1 版本資訊
description: Azure SDK for .NET 2.7 和 .NET 2.7.1 版本資訊
services: app-service\web
documentationcenter: .net
author: chrissfanos
editor: ''
ms.assetid: 877d070a-9bd5-49b3-8fac-6bb5f65c3554
ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 02/24/2017
ms.author: juliako
ms.openlocfilehash: 9a69253129cdedc4f5d7e736d5bd8d6a68f95a1e
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22986573"
---
# <a name="azure-sdk-for-net-27-and-net-271-release-notes"></a>Azure SDK for .NET 2.7 和 .NET 2.7.1 版本資訊
## <a name="overview"></a>概觀
本文件包含 Azure SDK for .NET 2.7 版的版本資訊。 

本文件也包含 Azure SDK for .NET 2.7.1 版的版本資訊。

只有在 Visual Studio 2015 和 Visual Studio 2013 中才支援 Azure SDK 2.7。 [Azure SDK 2.6](https://azure.microsoft.com/downloads/) 是 Visual Studio 2012 最後一個支援的 SDK。

如需此版本的詳細資訊，請參閱 [Azure SDK 2.7 公告文章](https://azure.microsoft.com/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)和 [Azure SDK 2.7.1 公告文章](http://go.microsoft.com/fwlink/?LinkId=623850)。

## <a name="azure-sdk-for-net-27"></a>Azure SDK for .NET 2.7
### <a name="sign-in-improvements-for-visual-studio-2015"></a>Visual Studio 2015 的登入增強功能
Visual Studio 2015 的 Azure SDK 2.7 支援 Visual Studio 2015 新的身分識別管理功能。  這包括支援透過角色型存取控制、雲端解決方案提供者、DreamSpark 以及其他帳戶和訂用帳戶類型存取 Azure 的帳戶。

Azure SDK 2.7 隨附的登入增強功能只有在 Visual Studio 2015 中才能使用。 Visual Studio 2013 的支援則包含於 Azure SDK 2.7.1 中。

### <a name="mobile-sdk"></a>Mobile SDK
已更新 **Mobile Apps** 範本，以反映最新的 [NuGet 封裝](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/) 和設定程序。

### <a name="service-bus"></a>服務匯流排
一般錯誤修正與增強功能。 如需更新和功能的詳細資訊，請參閱最新 [服務匯流排 NuGet](http://www.nuget.org/packages/WindowsAzure.ServiceBus/)的版本資訊。

### <a name="hdinsight-tools"></a>HDInsight 工具
此版本進行了下列更新。 這些更新目前為預覽版。 如需詳細資訊，請參閱 [此部落格](http://go.microsoft.com/fwlink/?LinkId=619108)。

* 為 Hive on Tez 工作繪製 Hive 圖形
* 完整的 Hive DML IntelliSense 支援
* Pig 範本支援
* Azure 服務的 Storm 範本

#### <a name="breaking-changes"></a>重大變更
* 使用這個版本的工具時，必須先升級舊的 **Storm** 專案。 如需詳細資訊，請參閱 [此部落格](http://go.microsoft.com/fwlink/?LinkId=619108)。
* 不再支援 Visual Studio Web Express。 如需詳細資訊，請參閱 [此部落格](http://go.microsoft.com/fwlink/?LinkId=619108)。

### <a name="azure-app-service-tools"></a>Azure App Service 工具
此版本對 Web 工具延伸模組進行了下列更新。 如需詳細資訊，請參閱[此部落格](https://azure.microsoft.com/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)。 

* 加入對 DreamSpark 帳戶的支援
* 對 Azure 工具進行全面的變更，以支援新的 Azure 資源管理 API
* 已在[雲端總管](#cloud_explorer)中新增對 Azure App Service 的支援

#### <a name="known-issues"></a>已知問題
伺服器總管的 [位置] 節點下方沒有出現 Web App 部署位置節點，而雲端總管下方沒有載入 Web App 部署位置的子節點。 這個問題已經解決，下一個 SDK 版本將不再發生。 

### <a name="cloud_explorer"></a>適用 Visual Studio 2015 的雲端總管
Azure SDK 2.7 包含適用 Visual Studio 2015 的雲端總管，可讓您從 Visual Studio 中檢視您的 Azure 資源、檢查其屬性，以及執行重要的開發人員動作。 

雲端總管支援下列功能：

* Azure 資源的 [資源群組] 檢視與 [資源類型] 檢視 
* 依名稱搜尋資源 (適用於 [資源類型] 檢視)
* 支援套用角色型存取控制 (RBAC) 的訂用帳戶與資源 
* 整合式的 [動作] 面板會針對特定選取的資源顯示以開發人員為焦點的動作。 例如：附加使用資源管理員堆疊建立之虛擬機器的遠端偵錯工具、檢視虛擬機器的診斷資料等等。
* 整合式的 [屬性] 面板會顯示以開發人員為焦點，在開發/測試期間經常需要的屬性 
* 快速切換列舉資源時要使用的帳戶 (使用工具列的 [設定] 命令) 
* 篩選列舉資源時要使用的訂用帳戶 (使用工具列的 [設定] 命令) 
* Azure 入口網站中用來管理資源與資源群組的深層連結 

### <a name="azure-resource-manager-tools"></a>Azure 資源管理員工具
Azure 資源管理員工具已更新為使用角色型存取控制 (RBAC) 和新的訂用帳戶類型。  除了傳統儲存體的功能之外，這些變更還附帶可使用新的儲存體帳戶在部署期間儲存成品。  

如果您使用舊版 SDK 的 Azure 資源群組專案搭配 SDK 2.7，必須使用新的儲存體帳戶，而不是傳統儲存體帳戶部署新的部署指令碼。  在變更您的專案以加入新的指令碼之前，會先提示您。  舊的指令碼將會重新命名，而且您必須手動修改新的指令碼。

### <a name="storage-explorer-tools"></a>儲存體總管工具
* 支援檢視附加 Blob。 如需詳細資訊，請參閱 [此部落格文章](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/04/13/introducing-azure-storage-append-blob.aspx)。 
* 支援透過伺服器總管檢視進階儲存體帳戶。 伺服器總管只會顯示進階儲存體帳戶的分頁 Blob，因為它們是進階儲存體帳戶唯一支援的類型。

### <a name="azure-data-factory-tools-for-visual-studio"></a>Visual Studio 的 Azure Data Factory 工具
介紹適用於 Visual Studio 的 **Azure Data Factory 工具** 。 以下是啟用的功能。 如需詳細資訊，請參閱 [此部落格](http://go.microsoft.com/fwlink/?LinkId=617530) 。

* **以範本為基礎進行撰寫**：選取以使用案例為基礎的範本、資料移動範本或資料處理範本，來部署端對端資料整合解決方案，並對 Data Factory 快速上手。 
* **整合方案總管，方便撰寫和部署 Data Factory 實體**：建立和部署管線與相關實體做為 Visual Studio 專案。 
* **整合圖表檢視，在撰寫同時也能進行視覺互動**：透過圖表檢視的協助，以視覺化方式撰寫管線與資料集。 
* **整合伺服器總管，方便瀏覽以及與已部署的實體互動**：利用伺服器總管，瀏覽已經部署的資料處理站及對應的實體。 將已部署的資料處理站或任何實體 (管線、連結的服務、資料集) 匯入您的專案。 
* **使用結構描述驗證與豐富的 IntelliSense 進行 JSON 編輯**：使用豐富的 IntelliSense 與結構描述驗證，有效率地設定和編輯 Data Factory 實體的 JSON 文件 
* **多重環境發佈**：藉由為每個環境建立不同的設定檔，將撰寫的管線發佈到開發、測試或生產環境。
* **支援以 Pig、Hive 和 .Net 為基礎的資料處理**：支援在 Data Factory 專案中使用 Pig 和 Hive 指令碼。 支援參考 C# 專案來管理 .Net 活動。

## <a name="azure-sdk-for-net-271"></a>Azure SDK for .NET 2.7.1
下節包含使用 Azure SDK for.NET 2.7.1 版本所導入的更新。

### <a name="hdinsight-tools"></a>HDInsight 工具
如需更多有關 HDInsight 工具更新的詳細說明，請參閱 [此部落格](http://go.microsoft.com/fwlink/?LinkId=623831)。

* Hive 工作運算子檢視 (新功能)
  
    為協助您更了解 Hive 查詢，因而新增了 Hive 運算子檢視功能。 若要查看頂點內的所有運算子，可按兩下工作圖形的頂點。 若要檢視特定運算子的更多詳細資訊，可將滑鼠停留在該運算子上方。
* Hive 錯誤標記 (新功能)
  
    為了讓您能夠立即檢視文法錯誤，因而新增了 Hive 錯誤標記功能。 此外，已增強錯誤訊息，您現在可以立即看到詳細的文法錯誤 (在此版本之前，您必須將 Hive 指令碼提交到叢集，並等候一些時間，才能取得有關錯誤訊息的詳細資料)。  
* Storm 拓撲圖表 (新功能)
  
    當您想要查看拓撲是否如預期般運作時，視覺化是非常重要的。 在這個版本中，我們新增了適用於 Storm 圖表的視覺效果。 您可以針對拓撲，將重要的度量資訊視覺化 (例如，某個色彩表示特定的閃電是否為「忙碌」狀態)。 您也可以按兩下閃電/水龍頭，來檢視更多詳細資料。
* 支援在 Azure 入口網站中建立的 HDInsight 叢集 (Bug 修正)
  
    您現在可以使用 Visual Studio 來檢視，並將工作提交到您的所有 HDInsight 叢集，而不論叢集建立於何處。
* 多個 IntelliSense 支援與更快速載入 Hive 中繼資料 (一個改進功能)
  
    我們已藉由新增更方便使用的建議來改進 IntelliSense。 例如，現在也建議在 IntelliSense 中使用資料表別名，讓您能夠更輕鬆地撰寫查詢。 此外，還改進了 Hive 中繼資料載入功能，只需數秒鐘，就能列出 Hive 中繼存放區的所有資料庫、資料表及資料行。

如需更多有關 HDInsight 工具更新的詳細說明，請參閱 [此部落格](http://go.microsoft.com/fwlink/?LinkId=623831)。

### <a name="improvements-in-visual-studio-2013"></a>Visual Studio 2013 的增強功能
* Azure SDK 2.7.1 讓 Visual Studio 2013 能夠透過角色型存取控制、雲端解決方案提供者和 Dreamspark 來存取 Azure 帳戶和訂用帳戶。
* 使用 Azure SDK 2.7.1，新的雲端總管工具視窗現在也可以在 Visual Studio 2013 中使用。

### <a name="known-issues"></a>已知問題
在非英文版的作業系統上安裝適用於 Visual Studio Community 2013 的 Azure SDK 2.6 或 2.7.1，將顯示警告來表示 Visual Studio 的英文版和非英文版資源可能不相符。 您可以安全地關閉此警告。 這只會發生在電腦未包含預先安裝的 Visual Studio Community 2013 且您正在非英文版的作業系統上安裝 SDK 時。 警告會在語言套件將 RTM 資源套用到 Visual Studio 之後，但在其套用 Update 4 之前顯示。 關閉警告將允許語言套件繼續執行並完成套用語言套件內容的 Update 4 版本。

LightSwitch 專案與這個版本不相容。 這個問題將在下個 SDK 版本中解決。

## <a name="also-see"></a>另請參閱
[Azure SDK 2.7.1 公告文章](http://go.microsoft.com/fwlink/?LinkId=623850)

[Azure SDK 2.7 公告文章](https://azure.microsoft.com/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)

[Azure SDK for .NET 和 API 的支援和停用資訊](https://msdn.microsoft.com/library/azure/dn479282.aspx/)

