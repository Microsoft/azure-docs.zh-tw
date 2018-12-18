---
title: Azure 資料處理站的疑難排解
description: 了解如何使用 Azure Data Factory 進行問題的疑難排解。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.assetid: 38fd14c1-5bb7-4eef-a9f5-b289ff9a6942
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: douglasl
robots: noindex
ms.openlocfilehash: cbd51c48ec5b1801062e7aaf0b77e7b347018b31
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051860"
---
# <a name="troubleshoot-data-factory-issues"></a>資料處理站的疑難排解
> [!NOTE]
> 本文適用於 Azure Data Factory 的第 1 版。 

這篇文章提供使用 Azure Data Factory 時的問題疑難排解提示。 這篇文章並未列出使用服務時的所有可能問題，但是涵蓋部分問題和一般疑難排解提示。   

## <a name="troubleshooting-tips"></a>疑難排解秘訣
### <a name="error-the-subscription-is-not-registered-to-use-namespace-microsoftdatafactory"></a>錯誤︰訂用帳戶未註冊為使用命名空間 'Microsoft.DataFactory'
如果您收到此錯誤，Azure Data Factory 資源提供者尚未在您的電腦上註冊。 執行下列動作：

1. 啟動 Azure PowerShell。
2. 使用下列命令來登入您的 Azure 帳戶。

    ```powershell
    Connect-AzureRmAccount
    ```
3. 執行下列命令以註冊 Azure Data Factory 提供者。

    ```powershell        
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

### <a name="problem-unauthorized-error-when-running-a-data-factory-cmdlet"></a>問題：執行 Data Factory Cmdlet 時發生未授權錯誤
您在 Azure PowerShell 中可能未使用正確的 Azure 帳戶或訂用帳戶。 請使用下列 Cmdlet 來選取要用於 Azure PowerShell 的正確 Azure 帳戶和訂用帳戶帳戶。

1. Connect-AzureRmAccount：使用正確的使用者識別碼和密碼
2. Get-AzureRmSubscription - 檢視帳戶的所有訂用帳戶。
3. Select-AzureRmSubscription &lt;訂用帳戶名稱&gt; - 選取正確的訂用帳戶。 請使用您在 Azure 入口網站上用來建立 Data Factory 的相同帳戶。

### <a name="problem-fail-to-launch-data-management-gateway-express-setup-from-azure-portal"></a>問題：無法從 Azure 入口網站啟動「資料管理閘道快速安裝」
資料管理閘道的快速安裝需要有 Internet Explorer 或 Microsoft ClickOnce 相容的 Web 瀏覽器。 如果無法啟動快速安裝，請執行下列其中一項：

* 使用 Internet Explorer 或 Microsoft ClickOnce 相容的 Web 瀏覽器。

    如果您使用 Chrome，請移至 [Chrome 線上應用程式商店](https://chrome.google.com/webstore/)，使用關鍵字 "ClickOnce" 進行搜尋，選擇其中一個 ClickOnce 擴充功能並安裝。

    針對 Firefox 進行相同的操作 (安裝附加元件)。 按一下工具列上的 [開啟功能表] 按鈕 (右上角的三條水平線)，按一下 [附加元件]，以「ClickOnce」關鍵字進行搜尋，選擇其中一個 ClickOnce 延伸模組並安裝。
* 使用入口網站中相同刀鋒視窗上顯示的 [手動安裝] 連結。 您可以使用這個方法來下載安裝檔案，然後手動執行它。 安裝成功之後，您會看到 [資料管理閘道組態] 對話方塊。 從入口網站畫面複製**金鑰**，並且在組態管理員中使用它來手動向服務註冊閘道器。  

### <a name="problem-fail-to-connect-to-on-premises-sql-server"></a>問題：無法連線到內部部署 SQL Server
在閘道器機器上啟動 [資料管理閘道組態管理員]，使用 [疑難排解] 索引標籤以測試從閘道器機器到 SQL Server 的連接。 如需連接/閘道器相關問題的疑難排解秘訣，請參閱 [針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) 。   

### <a name="problem-input-slices-are-in-waiting-state-for-ever"></a>問題：輸入配量永遠處於 Waiting 狀態
配量可能因各種原因而處於**等候中**狀態。 其中一個常見的原因是 **external** 屬性未設定為 **true**。 在 Azure Data Factory 範圍外產生的任何資料集，都應該標示 **external** 屬性。 此屬性可指出資料為外部資料，並未受到 Data Factory 內的任何管線支持。 一旦資料在個別的存放區可用，資料配量就會標示為 [就緒]。

關於 **external** 屬性的用法，請參閱下列範例。 當您將 external 設定為 true 時，可以視需要指定 **externalData**\*。

如需此屬性的詳細資訊，請參閱 [資料集](data-factory-create-datasets.md) 文章。

```json
{
  "name": "CustomerTable",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "MyLinkedService",
    "typeProperties": {
      "folderPath": "MyContainer/MySubFolder/",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": ";"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      }
    }
  }
}
```

若要解決這個錯誤，請將 **external** 屬性和選用 **externalData** 區段加入輸入資料表的 JSON 定義，並重新建立資料表。

### <a name="problem-hybrid-copy-operation-fails"></a>問題：混合式複製作業失敗
請參閱[針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshooting-gateway-issues)以取得對於使用資料管理閘道複製到/自內部部署資料存放區的問題的疑難排解步驟。

### <a name="problem-on-demand-hdinsight-provisioning-fails"></a>問題：隨選 HDInsight 佈建失敗
使用 HDInsightOnDemand 類型的連結服務時，您必須指定指向 Azure Blob 儲存體的 linkedServiceName。 Data Factory 服務會使用此儲存體來儲存記錄檔和您的隨選 HDInsight 叢集的支援檔案。  有時候隨選 HDInsight 叢集的佈建會失敗，並且有下列錯誤︰

```
Failed to create cluster. Exception: Unable to complete the cluster create operation. Operation failed with code '400'. Cluster left behind state: 'Error'. Message: 'StorageAccountNotColocated'.
```

此錯誤通常表示 linkedServiceName 中指定的儲存體帳戶的位置，與進行 HDInsight 佈建的資料中心位置不同。 範例︰如果您的 Data Factory 在美國西部，而 Azure 儲存體在美國東部，則隨選佈建會在美國西部發生失敗。

此外，還有第二個的 JSON 屬性 additionalLinkedServiceNames，可供隨選 HDInsight 中指定其他儲存體帳戶。 這些其他的連結儲存體帳戶應該與 HDInsight 叢集位於相同位置，否則會因相同的錯誤而發生失敗。

### <a name="problem-custom-net-activity-fails"></a>問題：自訂 .NET 活動失敗
如需詳細步驟，請參閱 [偵錯具有自訂活動的管線](data-factory-use-custom-activities.md#troubleshoot-failures) 。

## <a name="use-azure-portal-to-troubleshoot"></a>使用 Azure 入口網站進行疑難排解
### <a name="using-portal-blades"></a>使用入口網站刀鋒視窗
請參閱 [監視管線](data-factory-build-your-first-pipeline-using-editor.md#monitor-a-pipeline) 以取得步驟。

### <a name="using-monitor-and-manage-app"></a>使用監視及管理應用程式
如需詳細資訊，請參閱 [使用監視及管理應用程式，來監視及管理 Data Factory 管線](data-factory-monitor-manage-app.md) 。

## <a name="use-azure-powershell-to-troubleshoot"></a>使用 Azure PowerShell 進行疑難排解
### <a name="use-azure-powershell-to-troubleshoot-an-error"></a>使用 Azure PowerShell 對錯誤進行疑難排解
如需詳細資料，請參閱 [使用 Azure PowerShell 監視 Data Factory 管線](data-factory-build-your-first-pipeline-using-powershell.md#monitor-pipeline) 。

[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908
[cmdlet-reference]: http://go.microsoft.com/fwlink/?LinkId=517456
[json-scripting-reference]: http://go.microsoft.com/fwlink/?LinkId=516971

[azure-portal]: https://portal.azure.com/

[image-data-factory-troubleshoot-with-error-link]: ./media/data-factory-troubleshoot/DataFactoryWithErrorLink.png

[image-data-factory-troubleshoot-datasets-with-errors-blade]: ./media/data-factory-troubleshoot/DatasetsWithErrorsBlade.png

[image-data-factory-troubleshoot-table-blade-with-problem-slices]: ./media/data-factory-troubleshoot/TableBladeWithProblemSlices.png

[image-data-factory-troubleshoot-activity-run-with-error]: ./media/data-factory-troubleshoot/ActivityRunDetailsWithError.png

[image-data-factory-troubleshoot-dataslice-blade-with-active-runs]: ./media/data-factory-troubleshoot/DataSliceBladeWithActivityRuns.png

[image-data-factory-troubleshoot-walkthrough2-with-errors-link]: ./media/data-factory-troubleshoot/Walkthrough2WithErrorsLink.png

[image-data-factory-troubleshoot-walkthrough2-datasets-with-errors]: ./media/data-factory-troubleshoot/Walkthrough2DataSetsWithErrors.png

[image-data-factory-troubleshoot-walkthrough2-table-with-problem-slices]: ./media/data-factory-troubleshoot/Walkthrough2TableProblemSlices.png

[image-data-factory-troubleshoot-walkthrough2-slice-activity-runs]: ./media/data-factory-troubleshoot/Walkthrough2DataSliceActivityRuns.png

[image-data-factory-troubleshoot-activity-run-details]: ./media/data-factory-troubleshoot/Walkthrough2ActivityRunDetails.png
