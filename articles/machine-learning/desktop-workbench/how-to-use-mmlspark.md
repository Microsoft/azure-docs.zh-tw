---
title: 如何使用 MMLSpark Machine Learning | Microsoft Docs
description: 如何將 MMLSpark 程式庫與 Azure Machine Learning 搭配使用的指南。
services: machine-learning
author: rastala
ms.author: roastala
manager: jhubbard
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 01/12/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 6167e10219791466ca275ff02cc051e3227634e8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980029"
---
# <a name="how-to-use-microsoft-machine-learning-library-for-apache-spark"></a>如何使用適用於 Apache Spark 的 Microsoft Machine Learning 程式庫

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

## <a name="introduction"></a>簡介

[適用於 Apache Spark 的 Microsoft Machine Learning 程式庫](https://github.com/Azure/mmlspark) \(英文\) (MMLSpark) 提供工具，可讓您輕鬆地為大型資料集建立可調整的機器學習服務模型。 它包含將 SparkML 管線與 [Microsoft 辨識工具組](https://github.com/Microsoft/CNTK) \(英文\) 和 [OpenCV](http://www.opencv.org/) \(英文\) 整合，讓您能夠： 
 * 輸入與預先處理影像資料
 * 使用預先定型的深入學習模型來凸顯影像和文字
 * 使用隱含功能來為分類和迴歸模型定型和計分。

## <a name="prerequisites"></a>必要條件

若要逐步執行本作法指南，您需要：
- [安裝 Azure Machine Learning Workbench](quickstart-installation.md)
- [設定 Azure HDInsight Spark 叢集](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-jupyter-spark-sql)

## <a name="run-your-experiment-in-docker-container"></a>在 Docker 容器中執行您的實驗

您的 Azure Machine Learning Workbench 已設定為，當您在 Docker 容器中 (在本機或遠端 VM 中) 執行實驗時使用 MMLSpark。 這項功能可讓您輕鬆地使用 PySpark 模型進行偵錯和實驗，然後於叢集上按比例執行它們。 

若要開始使用範例，請建立新的專案，然後選取 [MMLSpark on Adult Census] (關於成人人口普查的 MMLSpark) 組件庫範例。 從專案儀表板中，選取 [Docker] 作為計算內容，然後按一下 [執行]。 Azure Machine Learning Workbench 會建置 Docker 容器來執行 PySpark 程式，然後執行程式。

完成執行之後，您可以在 Azure Machine Learning Workbench 的執行歷程記錄檢視中檢視結果。

## <a name="install-mmlspark-on-azure-hdinsight-spark-cluster"></a>在 Azure HDInsight Spark 叢集上安裝 MMLSpark。

若要完成此步驟和下一個步驟，您需要先[建立 Azure HDInsight Spark 叢集](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-jupyter-spark-sql)。

根據預設，Azure Machine Learning Workbench 會在您執行實驗時，於叢集上安裝 MMLSpark 封裝。 您可以藉由編輯專案資料夾中名為 _aml_config/spark_dependencies.yml_ 的檔案來控制此行為並安裝其他 Spark 封裝。

```
# Spark configuration properties.
configuration:
  "spark.app.name": "Azure ML Experiment"
  "spark.yarn.maxAppAttempts": 1

repositories:
  - "https://mmlspark.azureedge.net/maven"
packages:
  - group: "com.microsoft.ml.spark"
    artifact: "mmlspark_2.11"
    version: "0.9.9"
```

您也可以直接在 HDInsight Spark 叢集上使用[指令碼動作](https://github.com/Azure/mmlspark#hdinsight) \(英文\) 來安裝 MMLSpark。

## <a name="set-up-azure-hdinsight-spark-cluster-as-compute-target"></a>設定 Azure HDInsight Spark 叢集作為計算目標

從 Azure Machine Learning Workbench 移至 [檔案] 功能表，然後按一下 [開啟命令提示字元] 來開啟 CLI 視窗

在 CLI 視窗中，執行下列命令：

```
az ml computetarget attach cluster --name <myhdi> --address <myhdi-ssh.azurehdinsight.net> --username <sshusername> --password <sshpwd> 
```

```
az ml experiment prepare -c <myhdi>
```

現在叢集可用來做為專案的計算目標。

## <a name="run-experiment-on-azure-hdinsight-spark-cluster"></a>在 Azure HDInsight Spark 叢集上執行實驗。

返回 [MMLSpark on Adult Census] (關於成人人口普查的 MMLSpark) 範例的專案儀表板。 選取您的叢集作為計算目標，然後按一下 [執行]。

Azure Machine Learning Workbench 會將 Spark 作業提交至叢集。 您可以監視進度，並在執行歷程記錄檢視中檢視結果。

## <a name="next-steps"></a>後續步驟
如需 MMLSpark 程式庫的相關資訊，請參閱 [MMLSpark GitHub 存放庫](https://github.com/Azure/mmlspark) \(英文\)

Apache®、Apache Spark 和 Spark® 是 Apache Software Foundation 在美國及/或其他國家/地區的註冊商標或商標。
