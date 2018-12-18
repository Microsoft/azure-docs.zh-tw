---
title: 使用 Windows PC 搭配 Hadoop on HDInsight - Azure
description: 從 Hadoop on HDInsight 中的 Windows PC 作業。 使用 PowerShell、Visual Studio 和 Linux 工具來管理和查詢叢集。 使用 .NET 開發巨量資料解決方案。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.topic: conceptual
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 05/17/2017
ms.openlocfilehash: a3fbd9cca038547e92c034577042aebbac6fc1fe
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39593055"
---
# <a name="work-in-the-hadoop-ecosystem-on-hdinsight-from-a-windows-pc"></a>從 Windows PC 在 HDInsight 上的 Hadoop 生態系統中作業

深入了解 Windows 電腦上的開發和管理選項，以便在 HDInsight 上的 Hadoop 生態系統中作業。 

HDInsight 是以 Apache Hadoop 和 Hadoop 元件為基礎，採用在 Linux 上開發的開放原始碼技術。 HDInsight 3.4 版及更新版本會使用 Ubuntu Linux 發行版本作為叢集的基礎作業系統。 不過，您可以從 Windows 用戶端或 Windows 開發環境使用 HDInsight。

## <a name="use-powershell-for-deployment-and-management-tasks"></a>使用 PowerShell 進行部署和管理工作
Azure PowerShell 是一種指令碼環境，可讓您從 Windows 在 HDInsight 中用來控制及自動執行部署和管理工作。

您可以使用 PowerShell 執行的工作範例︰

* [使用 PowerShell 建立叢集](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
* [使用 PowerShell 執行 Hive 查詢](hadoop/apache-hadoop-use-hive-powershell.md)
* [使用 PowerShell 管理叢集](hdinsight-administer-use-powershell.md)

請遵循步驟來[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) 以取得最新的版本。 如果您需要修改指令碼才能使用適用於 Azure Resource Manager 的新 Cmdlet，請參閱[移轉至以 Azure Resource Manager 為基礎的開發工具 (適用於 HDInsight 叢集)](hdinsight-hadoop-development-using-azure-resource-manager.md)。

## <a name="utilities-you-can-run-in-a-browser"></a>您可以在瀏覽器中執行的公用程式
下列公用程式具有可在瀏覽器中執行的 Web UI：
* **[Azure Cloud Shell (預覽)](https://docs.microsoft.com/azure/cloud-shell/quickstart)** 是可在瀏覽器中以及從 Azure 入口網站執行的互動式、命令列殼層。
* **[Ambari Web UI](hdinsight-hadoop-manage-ambari.md)** 是 Azure 入口網站中可用的管理和監視公用程式，可用來管理不同種類的工作，例如︰
    * [使用 Ambari 搭配 REST API](hdinsight-hadoop-manage-ambari-rest-api.md)
    * [Ambari 中的 Hive 檢視](hadoop/apache-hadoop-use-hive-ambari-view.md)
    * [Ambari 中的 Tez 檢視](hdinsight-debug-ambari-tez-view.md)

## <a name="data-lake-hadoop-tools-for-visual-studio"></a>Data Lake (Hadoop) Tools for Visual Studio
使用 Data Lake Tools for Visual Studio 來部署和管理 Storm 拓撲。 Data Lake Tools 也會安裝 SCP.NET SDK，其可讓您使用 Visual Studio 開發 C# Storm 拓撲。

在您進行下列範例之前，請[安裝並嘗試 Data Lake Tools for Visual Studio](hadoop/apache-hadoop-visual-studio-tools-get-started.md)。 

您可以使用 Visual Studio 和 Data Lake Tools for Visual Studio 執行的工作範例︰
* [從 Visual Studio 部署和管理 Storm 拓撲](storm/apache-storm-deploy-monitor-topology-linux.md)
* [使用 Visual Studio 開發 Storm 的 C# 拓撲](storm/apache-storm-develop-csharp-visual-studio-topology.md)。 其中包含您可以連接到資料庫 (例如 Azure Cosmos DB 和 SQL Database) 之 Storm 拓撲的範例範本。

## <a name="visual-studio-and-the-net-sdk"></a>Visual Studio 和 .NET SDK 

您可以使用 Visual Studio 搭配 .NET SDK 來管理叢集和開發巨量資料應用程式。 您可以將其他 IDE 用於下列工作，但範例顯示在 Visual Studio 中。

您可以在 Visual Studio 中使用 .NET SDK 執行的工作範例︰
* [從 .NET Framework 應用程式在 HDInsight 中建立和使用叢集](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md)
* [使用 .NET SDK 執行 Hive 查詢](hadoop/apache-hadoop-use-hive-dotnet-sdk.md)
* [使用 C# 使用者定義的函式搭配 Hadoop 上的 Hive 和 Pig 串流](hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

> 提示：如果您執行的 .NET 方案具有以 Windows 為基礎的 HDInsight 叢集，現在是規劃移轉至以 Linux 為基礎的叢集的好時機。 如需詳細資訊，請參閱[將以 Windows 為基礎的 HDInsight 適用的 .NET 方案移轉至以 Linux 為基礎的 HDInsight](hdinsight-hadoop-migrate-dotnet-to-linux.md)。

## <a name="intellij-idea-and-eclipse-ide-for-spark-clusters"></a>適用於 Spark 叢集的 Intellij IDEA 和 Eclipse IDE
[Intellij IDEA](https://www.jetbrains.com/idea/download) 和 [Eclipse IDE](https://www.eclipse.org/downloads/) 都可以用來︰
* 在 HDInsight Spark 叢集上開發並提交 Scala Spark 應用程式。
* 存取 Spark 叢集資源。
* 在本機開發並執行 Scala Spark 應用程式。

這些文章顯示如何︰ 
* Intellij IDEA︰[使用 Azure Toolkit for Intellij 外掛程式和 Scala SDK 建立 Spark 應用程式](spark/apache-spark-intellij-tool-plugin.md)。
* Eclipse IDE 或 Scala IDE for Eclipse：[建立 Spark 應用程式和 Azure Toolkit for Eclipse](spark/apache-spark-eclipse-tool-plugin.md) 


## <a name="notebooks-on-spark-for-data-scientists"></a>適用於資料科學家的 Spark Notebook 
HDInsight 中的 Apache Spark 叢集包含可搭配 Jupyter Notebook 使用的 Zeppelin Notebook 和核心。 

* [了解如何使用 Spark 叢集上的核心搭配 Jupyter Notebook 來測試 Spark 應用程式](spark/apache-spark-zeppelin-notebook.md)
* [了解如何使用 Spark 叢集上的 Zeppelin Notebook 來執行 Spark 作業](spark/apache-spark-jupyter-notebook-kernels.md) 


## <a name="run-linux-based-tools-and-technologies-on-windows"></a>在 Windows 上執行以 Linux 為基礎的工具和技術

如果您遇到必須使用只適用於 Linux 之工具或技術的情況，請考慮下列選項︰

* **Windows 10 上的 Bash (Beta 版)** 在 Windows 上提供 Linux 子系統。 Bash 可讓您直接執行 Linux 公用程式，而不必維護專用的 Linux 安裝。 [在 Windows 10 上安裝和執行 Bash Beta 版](https://msdn.microsoft.com/commandline/wsl/install_guide)
* **Docker for Windows** 可供存取許多以 Linux 為基礎的工具，並可以直接從 Windows 執行。 例如，您可以使用 Docker 直接從 Windows 執行 Hive 適用的 Beeline 用戶端。 您也可以使用 Docker 來執行本機 Jupyter Notebook，並從遠端連線到 HDInsight 上的 Spark。 [開始使用 Docker for Windows](https://docs.docker.com/docker-for-windows/)
* **[MobaXTerm](http://mobaxterm.mobatek.net/)** 可讓您透過 SSH 連線，以圖形方式瀏覽叢集檔案系統。

## <a name="next-steps"></a>後續步驟
如果您不熟悉使用以 Linux 為基礎的叢集，請參閱下列文章︰
* [設定 Hadoop、Kafka、Spark 或其他叢集](hdinsight-hadoop-provision-linux-clusters.md)
* [Linux 上的 HDInsight 叢集祕訣](hdinsight-hadoop-linux-information.md)
