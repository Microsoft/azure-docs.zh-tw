---
title: 從 Azure Blob 儲存體來回移動資料 | Microsoft Docs
description: 從 Azure Blob 儲存體來回移動資料
services: machine-learning,storage
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: d6681e30-ab45-45ea-a9fb-ac8acefe544d
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: 717fdd2053cae28234458e197f8211ef25cf7f9d
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394386"
---
# <a name="move-data-to-and-from-azure-blob-storage"></a>從 Azure Blob 儲存體來回移動資料
[!INCLUDE [cap-ingest-data-selector](../../../includes/cap-ingest-data-selector.md)]

<!-- just in case, adding this to separate these two include references -->

[!INCLUDE [blob-storage-tool-selector](../../../includes/machine-learning-blob-storage-tool-selector.md)]

哪一種方法最適合會取決於您的案例。 [在 Azure 機器學習中的進階分析案例](plan-sample-scenarios.md) 文章可協助您判斷進階分析程序中各種資料科學工作流程所需的資源。

> [!NOTE]
> 如需 Azure Blob 儲存體的完整介紹，請參閱 [Azure Blob 基本概念](../../storage/blobs/storage-dotnet-how-to-use-blobs.md)和 [Azure Blob 服務](https://msdn.microsoft.com/library/azure/dd179376.aspx)。
> 
> 

或者，您可以使用 [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) 來︰ 

* 建立和排程管線，以從 Azure Blob 儲存體下載資料， 
* 將其傳遞至已發佈的 Azure Machine Learning Web 服務， 
* 接收預測性分析結果，並 
* 將結果上傳至儲存體。 

如需詳細資訊，請參閱 [使用 Azure Data Factory 和 Azure Machine Learning 來建立預測管線](../../data-factory/transform-data-using-machine-learning.md)。

## <a name="prerequisites"></a>必要條件
本文件假設您擁有 Azure 訂用帳戶、儲存體帳戶和該帳戶的對應儲存體金鑰。 上傳/下載資料之前，您必須知道 Azure 儲存體帳戶名稱和帳戶金鑰。

* 若要設定 Azure 訂用帳戶，請參閱 [免費試用一個月](https://azure.microsoft.com/pricing/free-trial/)。
* 如需建立儲存體帳戶的指示，以及取得帳戶和金鑰的資訊，請參閱 [關於 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md)。

