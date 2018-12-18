---
title: 關於 Batch AI 資源 - Azure | Microsoft Docs
description: Microsoft Azure Batch AI 服務中工作區、叢集、檔案伺服器、實驗和作業的概觀。
services: batch-ai
documentationcenter: na
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.custom: ''
ms.service: batch-ai
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 08/15/2018
ms.author: danlep
ms.openlocfilehash: 4a9e3529f9d68ecdc614ea69cffc6897891f4548
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44056701"
---
# <a name="overview-of-resources-in-batch-ai"></a>Batch AI 中資源的概觀

第一次開始使用 Batch AI 服務時，建議您先了解可用的 Batch AI 資源。 和其他 Azure 服務一樣，您可以在一或多個 Azure「資源群組」中建立 Batch AI 資源。 在資源群組中建立一或多個 Batch AI「工作區」。 每個工作區都包含混合的 Batch AI「叢集」、「檔案伺服器」和「實驗」。 Batch AI 實驗會封裝一個群組的「作業」。

下圖顯示 Batch AI 的資源階層範例。 

![](./media/migrate-to-new-api/batch-ai-resource-hierarchy.png)

下列各節會深入探討 Batch AI 資源。

## <a name="workspace"></a>工作區

Batch AI 中的工作區是其餘 Batch AI 資源的最上層集合。 工作區有助於將屬於不同群組或專案的工作分開。 例如，您可以分別建立開發和測試工作區。

## <a name="cluster"></a>叢集

Batch AI 中的叢集包含了用於執行作業的計算資源。 叢集中的所有節點都有相同的 VM 大小和 OS 映像。 Batch AI 提供許多選項供您建立自訂叢集以因應不同需求。 一般來說，您會對要完成專案所需的每個處理能力類別設定不同的叢集。 請根據需求和預算，來相應增加和減少 Batch AI 叢集。 如需詳細資訊，請參閱[使用 Batch AI 叢集](clusters.md)。

## <a name="file-server"></a>檔案伺服器

您可以選擇在 Batch AI 中建立檔案伺服器，以儲存資料、訓練指令碼和輸出記錄。 Batch AI 檔案伺服器是受控的單一節點 NFS，可自動裝載到叢集節點上，為作業提供可輕鬆且集中存取的儲存體位置。 在大部分情況下，工作區中只需要一個檔案伺服器，而且您可以將訓練作業的資料分到不同目錄中。 如果 NFS 不適用於工作負載，Batch AI 還可支援其他儲存體選項，包括 [Azure 儲存體](use-azure-storage.md)或自訂解決方案，例如 Gluster 或 Lustre 檔案系統。

## <a name="experiment"></a>實驗

實驗會將您所查詢和管理的一組相關作業群組在一起。 每個工作區可有多次實驗，而每個實驗都會嘗試解決一個特定問題。

## <a name="job"></a>工作 (Job)

作業是需要執行的單一工作或指令碼，例如，為了訓練深度學習模型而執行。 每個作業都會在工作區中的一個叢集上執行特定指令碼  (此指令碼可儲存在 Batch AI 檔案伺服器上，也可儲存在其他儲存體解決方案上)。每個 Batch AI 作業都有相關聯的架構類型：TensorFlow、Horovod、CNTK、Caffe、Caffe2、pyTorch、Chainer、自訂 MPI 或自訂。 針對每個架構，Batch AI 服務會設定必要的基礎結構並管理作業程序。 每個實驗都可以有相似、但對於不同參數有些許變更的多個作業。

## <a name="next-steps"></a>後續步驟

* 執行您的第一個 [Batch AI 訓練作業](quickstart-tensorflow-training-cli.md)。

* 查看不同架構的範例[訓練配方](https://github.com/Azure/BatchAI/tree/master/recipes)。