---
title: 從 Apache Spark 應用程式 Azure HDInsight RequestBodyTooLarge 錯誤
description: NativeAzureFileSystem ...RequestBodyTooLarge 會在記錄檔中顯示 Apache Spark 串流應用程式 Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: 73ae646cb083841ee1d55b6c7ce6af7180cef08e
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98929422"
---
# <a name="nativeazurefilesystemrequestbodytoolarge-appear-in-apache-spark-streaming-app-log-in-hdinsight"></a>"NativeAzureFileSystem...RequestBodyTooLarge "會顯示在 HDInsight 的 Apache Spark 串流應用程式記錄檔中

本文說明在 Azure HDInsight 叢集中使用 Apache Spark 元件時，疑難排解步驟和可能的解決方案。

## <a name="issue"></a>問題

錯誤： `NativeAzureFileSystem ... RequestBodyTooLarge` 會出現在 Apache Spark 串流應用程式的驅動程式記錄檔中。

## <a name="cause"></a>原因

您的 Spark 事件記錄檔可能達到 WASB 的檔案長度限制。

在 Spark 2.3 中，每個 Spark 應用程式都會產生一個 Spark 事件記錄檔。 Spark 串流應用程式的 Spark 事件記錄檔會在應用程式執行時持續成長。 現在 WASB 上的檔案具有50000區塊限制，而預設區塊大小為 4 MB。 因此，在預設設定中，檔案大小上限為 195 GB。 不過，Azure 儲存體已將最大區塊大小增加到 100 MB，這可有效地將單一檔案限制為 4.75 TB。 如需詳細資訊，請參閱 [Blob 儲存體的延展性和效能目標](../../storage/blobs/scalability-targets.md)。

## <a name="resolution"></a>解決方案

此錯誤有三個可用的解決方案：

* 將區塊大小增加到最多 100 MB。 在 Ambari UI 中，修改 HDFS 設定屬性 `fs.azure.write.request.size` (或在) 區段中加以建立 `Custom core-site` 。 將屬性設為較大的值，例如：33554432。 儲存更新的設定，並重新啟動受影響的元件。

* 定期停止並重新提交 spark 串流作業。

* 使用 HDFS 來儲存 Spark 事件記錄檔。 使用 HDFS 進行儲存體可能會導致叢集調整或 Azure 升級期間遺失 Spark 事件資料。

    1. 透過 `spark.eventlog.dir` `spark.history.fs.logDirectory` Ambari UI 進行變更：

        ```
        spark.eventlog.dir = hdfs://mycluster/hdp/spark2-events
        spark.history.fs.logDirectory = "hdfs://mycluster/hdp/spark2-events"
        ```

    1. 在 HDFS 上建立目錄：

        ```
        hadoop fs -mkdir -p hdfs://mycluster/hdp/spark2-events
        hadoop fs -chown -R spark:hadoop hdfs://mycluster/hdp
        hadoop fs -chmod -R 777 hdfs://mycluster/hdp/spark2-events
        hadoop fs -chmod -R o+t hdfs://mycluster/hdp/spark2-events
        ```

    1. 透過 Ambari UI 重新開機所有受影響的服務。

## <a name="next-steps"></a>後續步驟

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]