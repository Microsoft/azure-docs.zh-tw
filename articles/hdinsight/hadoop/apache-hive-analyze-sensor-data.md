---
title: 使用 Hive 及 Hadoop 分析感應器資料 - Azure HDInsight
description: 了解如何使用 Hive 查詢主控台搭配 HDInsight (Hadoop) 分析感應器資料，然後在 Microsoft Excel 中使用 Power View 將資料視覺化。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 04/14/2017
ROBOTS: NOINDEX
ms.openlocfilehash: f82bac1b478183cad41e1bb9f7dce3fed8b5417b
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048893"
---
# <a name="analyze-sensor-data-using-the-hive-query-console-on-hadoop-in-hdinsight"></a>在 HDInsight 的 Hadoop 上使用 Hive 查詢主控台分析感應器資料

了解如何使用 Hive 查詢主控台搭配 HDInsight (Hadoop) 分析感應器資料，然後在 Microsoft Excel 中使用 Power View 將資料視覺化。

> [!IMPORTANT]
> 本文件的步驟只適用於 Windows HDInsight 叢集。 Windows 上的 HDInsight 只提供低於 HDInsight 3.4 的版本。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。


在此範例中，您會使用 Hive 來處理歷程記錄資料，並找出暖氣及空調系統的問題。 具體而言，您會執行下列工作，找出無法穩定維持所設定之溫度的系統：

* 建立 Hive 資料表來查詢逗點分隔值 (CSV) 檔案中儲存的資料。
* 建立 Hive 查詢以分析資料。
* 若要擷取已分析的資料，請使用 Microsoft Excel 連接到 HDInsight。
* 若要將資料視覺化，請使用 Power View。

![A diagram of the solution architecture](./media/apache-hive-analyze-sensor-data/hvac-architecture.png)

## <a name="prerequisites"></a>必要條件

* HDInsight (Hadoop) 叢集：如需建立叢集的相關資訊，請參閱 [在 HDInsight 中建立 Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。
* Microsoft Excel 2013

  > [!NOTE]
  > Microsoft Excel 用於搭配 [Power View](https://support.office.com/Article/Power-View-Explore-visualize-and-present-your-data-98268d31-97e2-42aa-a52b-a68cf460472e?ui=en-US&rs=en-US&ad=US)進行資料視覺化。

* [Microsoft Hive ODBC 驅動程式](http://www.microsoft.com/download/details.aspx?id=40886)

## <a name="to-run-the-sample"></a>執行範例

1. 請使用網頁瀏覽器瀏覽至下列 URL： 

         https://<clustername>.azurehdinsight.net

    將 `<clustername>` 替換為 HDInsight 叢集的名稱。

    出現提示時，使用您佈建此叢集時所用的系統管理員使用者名稱和密碼來進行驗證。

2. 從開啟的網頁中，按一下 [Getting Started Gallery] 索引標籤，然後在 [搭配範例資料的方案] 類別下，按一下 [感應器資料分析] 範例。

    ![開始使用資源庫映像](./media/apache-hive-analyze-sensor-data/getting-started-gallery.png)

3. 依照網頁上提供的指示完成範例。
