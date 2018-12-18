---
title: Hadoop 架構 - Azure HDInsight
description: 說明 HDInsight 叢集上的 Hadoop 儲存體和處理工作。
services: hdinsight
author: ashishthaps
ms.author: ashishth
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/19/2018
ms.openlocfilehash: f22cb6a56e0ef81e3d7799b38e33113f8b175457
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699425"
---
# <a name="hadoop-architecture-in-hdinsight"></a>HDInsight 上的 Hadoop 架構

Hadoop 包含兩個核心元件：提供儲存體的 Hadoop 分散式檔案系統 (HDFS)，和提供處理功能的 Yet Another Resource Negotiator (YARN)。 利用儲存體和處理功能，叢集變成能夠執行 MapReduce 程式以進行所需的資料處理。

> [!NOTE]
> 通常不會在 HDInsight 叢集內部署 HDFS 來提供儲存體。 相反地，Hadoop 元件使用 HDFS 相容的介面層。 實際的儲存體功能是由 Azure 儲存體或 Azure Data Lake Store 提供。 對於 Hadoop，在 HDInsight 叢集上執行的 MapReduce 作業會像 HDFS 存在一般運作，因此不需要變更即可支援其儲存需求。 在 HDInsight 上的 Hadoop 中，儲存體是外部的，但 YARN 處理仍然是核心元件。 如需詳細資訊，請參閱 [Azure HDInsight 簡介](hadoop/apache-hadoop-introduction.md)。

本文會介紹 YARN 以及 YARN 是如何協調 HDInsight 上應用程式的執行工作。

## <a name="yarn-basics"></a>YARN 基本概念 

YARN 掌管並協調 Hadoop 中的資料處理。 YARN 有兩種核心服務，在叢集節點上作為流程執行： 

* ResourceManager 
* NodeManager

ResourceManager 會將叢集的計算資源授與 MapReduce 作業這類應用程式。 ResourceManager 以容器的方式提供這類資源，每個容器的組成即是 CPU 核心和 RAM 記憶體的分配。 如果您結合叢集中所有可用的資源，然後將核心和記憶體分配給數個區塊，則每個資源區塊就是一個容器。 叢集中的每個節點有容量可容納一定數量的容器，因此叢集可用的容器數目有固定的限制。 容器中的資源配額可以設定。 

在叢集上執行 MapReduce 應用程式時，ResourceManager 會提供要在其中執行應用程式的容器。 ResourceManager 會追蹤執行應用程式的狀態、可用的叢集容量，也會追蹤應用程式執行完成的時刻，並將資源釋出。 

ResourceManager 也會執行 Web 伺服器處理序，提供可用來監控應用程式狀態的 Web 使用者介面。

當使用者提交要在叢集上執行的 MapReduce 應用程式時，會將應用程式提交至 ResourceManager。 接著，ResourceManager 會在可用的 NodeManager 節點上配置容器。 NodeManager 節點是應用程式實際執行的地方。 第一個配置的容器會執行名為 ApplicationMaster 的特殊應用程式。 此 ApplicationMaster 負責以後續容器的形式，取得執行提交之應用程式所需的資源。 ApplicationMaster 會檢查應用程式的階段 (例如，對應階段和簡化階段)，並將資料的需求量納入考量。 接著，ApplicationMaster 會代表應用程式向 ResourceManager 要求 (*交涉*) 資源。 ResourceManager 再將叢集中 NodeManagers 的資源授與 ApplicationMaster，供其在執行應用程式時使用。 

NodeManagers 會執行組成應用程式的各項工作，然後將其進度和狀態回報給 ApplicationMaster。 ApplicationMaster 再將應用程式的狀態回報給 ResourceManager。 ResourceManager 會將任何結果傳回用戶端。

## <a name="yarn-on-hdinsight"></a>HDInsight 上的 YARN

所有的 HDInsight 叢集類型皆會部署 YARN。 ResourceManager 的部署是為了達到高可用性，有主要和次要執行個體，其分別在叢集內的第一個和第二個前端節點上執行。 一次只會有一個 ResourceManager 執行個體處於使用中狀態。 NodeManager 執行個體會在叢集中可用的背景工作節點上執行。

![HDInsight 上的 YARN](./media/hdinsight-hadoop-architecture/yarn-on-hdinsight.png)

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 上的 Hadoop 中使用 MapReduce](hadoop/hdinsight-use-mapreduce.md)
* [Azure HDInsight 簡介](hadoop/apache-hadoop-introduction.md)
