---
title: 在 HDInsight 中搭配使用 Hadoop Pig 與 PowerShell - Azure
description: 了解如何使用 Azure PowerShell 將 Pig 工作提交至 HDInsight 上的 Hadoop 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/09/2018
ms.author: jasonh
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.openlocfilehash: ff08e632c1bfd8eb4040e4e746ce08335eba8b08
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047369"
---
# <a name="use-azure-powershell-to-run-pig-jobs-with-hdinsight"></a>使用 Azure PowerShell 執行 Pig 工作與 HDInsight

[!INCLUDE [pig-selector](../../../includes/hdinsight-selector-use-pig.md)]

本文件提供使用 Azure PowerShell 將 Pig 工作提交至 HDInsight 叢集上的 Hadoop 的範例。 Pig 可讓您使用可建立資料轉換模型的語言 (Pig Latin) 撰寫 MapReduce 工作，而不是撰寫對應和歸納函數。

> [!NOTE]
> 本文件不提供範例中使用的 Pig Latin 陳述式所執行的工作詳細的描述。 如需此範例中使用的 Pig Latin 相關資訊，請參閱 [在 HDInsight 上搭配 Hadoop 使用 Pig](hdinsight-use-pig.md)。

## <a id="prereq"></a>必要條件

* **Azure HDInsight 叢集**

  > [!IMPORTANT]
  > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* **具有 Azure PowerShell 的工作站**。

## <a id="powershell"></a>執行 Pig 作業

Azure PowerShell 提供 *Cmdlet* ，可讓您從遠端在 HDInsight 上執行 Pig 工作。 就內部而言，PowerShell 會使用 REST 呼叫 HDInsight 叢集上執行的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat)。

在遠端 HDInsight 叢集上執行 Pig 工作時，會使用下列 Cmdlet：

* **Connect-AzureRmAccount**：向您的 Azure 訂用帳戶驗證 Azure PowerShell。
* **New-AzureRmHDInsightPigJobDefinition**：使用指定的 Pig Latin 陳述式建立「作業定義」。
* **Start-AzureRmHDInsightJob**：將作業定義傳送到 HDInsight 並開始作業。 系統會傳回「作業」物件。
* **Wait-AzureRmHDInsightJob**：使用工作物件來檢查工作的狀態。 它會等到作業完成，或超過等候時間。
* **Get-AzureRmHDInsightJobOutput**：用來擷取工作的輸出。

下列步驟示範如何使用這些 Cmdlet，在您的 HDInsight 叢集上執行工作。

1. 使用編輯器，將下列程式碼儲存為 **pigjob.ps1**。

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-pig/use-pig.ps1?range=5-51)]

1. 開啟新的 Windows PowerShell 命令提示字元。 將目錄變更至 **pigjob.ps1** 檔案的位置，然後使用下列命令來執行指令碼：

        .\pigjob.ps1

    系統會提示您登入 Azure 訂用帳戶。 接著，您必須提供 HDInsight 叢集的 HTTP/管理帳戶名稱和密碼。

2. 作業完成時，應該會傳回與下面文字類似的資訊：

        Start the Pig job ...
        Wait for the Pig job to complete ...
        Display the standard output ...
        (TRACE,816)
        (DEBUG,434)
        (INFO,96)
        (WARN,11)
        (ERROR,6)
        (FATAL,2)

## <a id="troubleshooting"></a>疑難排解

如果作業完成時未傳回任何資訊，請檢視錯誤記錄。 若要檢視這項工作的錯誤資訊，請將下列命令新增至 **pigjob.ps1** 檔案的結尾，並儲存它，然後重新予以執行。

    # Print the output of the Pig job.
    Write-Host "Display the standard error output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $pigJob.JobId `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

這個 Cmdlet 會傳回作業處理期間寫入到 STDERR 的資訊。

## <a id="summary"></a>摘要
如您所見，Azure PowerShell 提供簡單的方法，在 HDInsight 叢集上執行 Pig 工作、監視工作狀態，以及擷取輸出。

## <a id="nextsteps"></a>後續步驟
如需 HDInsight 中 Pig 的一般資訊：

* [搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)

如需您可以在 HDInsight 上使用 Hadoop 之其他方式的詳細資訊：

* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)
* [搭配使用 MapReduce 與 HDInsight 上的 Hadoop](hdinsight-use-mapreduce.md)
