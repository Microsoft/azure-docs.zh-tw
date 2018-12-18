---
title: 何謂 Azure Databricks？ | Microsoft Docs
description: 了解何謂 Azure Databricks，以及它如何將 Databricks 上的 Spark 帶入 Azure。 Azure Databricks 是一個針對 Microsoft Azure 雲端服務平台進行最佳化的 Apache Spark 分析平台。
services: azure-databricks
documentationcenter: ''
author: nitinme
manager: cgronlun
editor: cgronlun
ms.service: azure-databricks
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 05/29/2018
ms.author: nitinme
ms.custom: mvc
ms.openlocfilehash: c621962c8ff0dcdb5070a81c5732012cb0898394
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34599733"
---
# <a name="what-is-azure-databricks"></a>何謂 Azure Databricks？

Azure Databricks 是一個針對 Microsoft Azure 雲端服務平台進行最佳化的 Apache Spark 分析平台。 Databricks 由 Apache Spark 的創立者所設計，可與 Azure 整合，提供一鍵式設定、順暢的工作流程以及互動式的工作區，可讓資料科學家、資料工程師及企業分析師共同作業。

![何謂 Azure Databricks？](./media/what-is-azure-databricks/azure-databricks-overview.png "何謂 Azure Databricks？")

## <a name="apache-spark-based-analytics-platform"></a>以 Apache Spark 為基礎的分析平台

Azure Databricks 包含完整的開放原始碼 Apache Spark 叢集技術和功能。 Azure Databricks 中的 Spark 包括下列元件：

![Azure Databricks 中的 Apache Spark](./media/what-is-azure-databricks/apache-spark-ecosystem-databricks.png "Azure Databricks 中的 Apache Spark")

* **Spark SQL 和 DataFrame**：Spark SQL 是用於使用結構化資料的 Spark 模組。 DataFrame 則是組織成具名資料行的分散式資料集合。 其概念等同於關聯式資料庫中的資料表或 R/Python 中的資料框架。

* **資料流**：分析應用程式和互動式應用程式的即時資料處理和分析。 與 HDFS、Flume 和 Kafka 整合。

* **MLib**：一種機器學習程式庫，由常見的學習演算法和公用程式 (包括分類、迴歸、群集、協同篩選、維度縮減，以及基礎最佳化基本項目) 所組成。

* **GraphX**：圖表和圖表計算，適用於廣泛的使用案例範圍，從認知分析到資料探索。

* **Spark Core API**：包括 R、SQL、Python、Scala 和 Java 的支援。

## <a name="apache-spark-in-azure-databricks"></a>Azure Databricks 中的 Apache Spark

Azure Databricks 是以 Spark 的功能為基礎所建置，其方法是藉由提供包含下列各項的零管理雲端平台：

- 完全受控的 Spark 叢集
- 適用於探索和視覺效果的互動式工作區
- 為您最愛的 Spark 架構應用程式賦予能力的平台

### <a name="fully-managed-apache-spark-clusters-in-the-cloud"></a>雲端中完全受控的 Apache Spark 叢集

Azure Databricks 在雲端中擁有安全而可靠的生產環境，並由 Spark 專家管理和支援。 您可以：

* 在數秒鐘內建立叢集。
* 動態地自動擴大和縮減叢集規模 (包括無伺服器叢集)，並與所有小組分享這些叢集。 
* 透過 REST API，以程式設計方式使用叢集。 
* 使用以 Spark 為基礎所建置的安全資料整合功能，讓您不必集中資料就可予以整合。 
* 立即存取每個版本的最新 Apache Spark 功能。

### <a name="databricks-runtime"></a>Databricks 執行階段
Databricks 執行階段是以 Apache Spark 為基礎所建置，並且原生就是針對 Azure 雲端所建置的。 

透過**無伺服器**選項，Azure Databricks 可完全去除基礎結構的複雜性，讓您不必擁有特殊的專業技術，就能安裝和設定資料基礎結構。 無伺服器選項可協助資料科學家猶如團隊一般地快速逐一查看。

至於重視生產作業效能的資料工程師，Azure Databricks 則提供 Spark 引擎，其可透過各種最佳化在 I/O 層和處理層 (Databricks I/O) 獲得較快的速度和較優異的效能。

### <a name="workspace-for-collaboration"></a>共同作業的工作區

透過共同作業且整合的環境，Azure Databricks 可簡化資料探索、建立原型和在 Spark 中執行資料導向應用程式的程序。

* 決定如何利用簡單的資料探索來使用資料。
* 在 Notebook 中以 R、Python、Scala 或 SQL 記載進度。
* 點選幾下即可將資料視覺化，並使用熟悉的工具，例如 Matplotlib、ggplot 或 d3。
* 使用互動式儀表板來建立動態報告。
* 使用 Spark，並同時與資料互動。

## <a name="enterprise-security"></a>企業安全性

Azure Databricks 可提供企業級的 Azure 安全性，包括 Azure Active Directory 整合、角色型控制，以及可保護您的資料和業務的 SLA。

* 與 Azure Active Directory 整合可讓您使用 Azure Databricks 執行完整的 Azure 型解決方案。
* Azure Databricks 的角色型存取可讓您針對 Notebook、叢集、作業和資料提供更細緻的使用者權限。
* 企業級 SLA。 

## <a name="integration-with-azure-services"></a>Azure 服務整合

Azure Databricks 與 Azure 的資料庫和儲存體有深入的整合：SQL 資料倉儲、Cosmos DB、Data Lake Store 和 Blob 儲存體。 

## <a name="integration-with-power-bi"></a>Power BI 整合
透過 Power BI 的豐富整合，Azure Databricks 可讓您快速且輕鬆地探索和分享具有影響力的深入解析。 您也可以使用其他 BI 工具，例如透過 JDBC/ODBC 叢集端點使用 Tableau Software。

## <a name="next-steps"></a>後續步驟

* [快速入門：在 Azure Databricks 執行 Spark 作業](quickstart-create-databricks-workspace-portal.md)
* [使用 Spark 叢集](https://docs.azuredatabricks.net/user-guide/clusters/index.html)
* [使用 Notebook](https://docs.azuredatabricks.net/user-guide/notebooks/index.html)
* [建立 Spark 作業](https://docs.azuredatabricks.net/user-guide/jobs.html)

 









