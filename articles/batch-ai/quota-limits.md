---
title: Azure Batch AI 的服務配額和限制 | Microsoft Docs
description: 了解預設的 Azure Batch AI 配額、限制和條件約束，以及如何要求增加配額
services: batch-ai
documentationcenter: ''
author: johnwu10
manager: jeconnoc
editor: ''
ms.service: batch-ai
ms.topic: article
ms.date: 08/08/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: cade124cefbd4e2e63ab4cb6fa4f22b3bd672ad0
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391805"
---
# <a name="batch-ai-service-quotas-and-limits"></a>Batch AI 服務配額和限制

如同使用其他 Azure 服務，對於與 Batch AI 服務相關聯的特定資源有一些限制。 在 Batch AI 中，限制就是套用在[可提供](https://azure.microsoft.com/global-infrastructure/services/)服務的各區域訂用帳戶層級上的預設配額。 本文將討論這些預設值，以及如何申請加大配額。

當您設計和相應增加您的 Batch AI 資源時，請記住這些配額。 例如，如果您的叢集未達到您所指定的目標節點數目，您可能已達到訂用帳戶的 Batch AI 核心限制。

如果您計劃在 Batch AI 中執行生產工作負載，您可能需要增加一或多個高於預設值的配額。

> [!NOTE]
> 配額是一種信用限制，不是容量保證。 如果您有大規模的容量需求，請連絡 Azure 支援。
> 
> 

## <a name="resource-quotas"></a>資源配額

[!INCLUDE [azure-batch-ai-limits](../../includes/azure-batch-ai-limits.md)]

## <a name="other-limits"></a>其他限制

以下是不可超過一次叫用的嚴格限制。

| **Resource** | **上限** |
| --- | --- |
| 每個資源群組的最大工作區 | 800 |
| 叢集大小上限 | 100 個節點 |
| 每個節點的最大 GPU MPI 流程數 | 1 - 4 |
| 每個節點的最大 GPU 背景工作角色數 | 1 - 4 |
| 作業存留期上限 | 7 天<sup>1</sup> |
| 每個節點的最大參數伺服器數 | 1 |

<sup>1</sup> 最長存留期指的是作業開始執行到完成的時間。 已完成的作業會無限期保留；無法存取未在最長存留期內完成之作業的資料。

## <a name="view-batch-ai-quotas"></a>檢視 Batch AI 配額

在 [Azure 入口網站][portal]中檢視目前的 Batch AI 訂用帳戶配額。

1. 在左側窗格中按一下 [所有服務]。 然後搜尋 **Batch AI**，按一下以開啟服務。
2. 按一下 Batch AI 功能表中的 [使用量 + 配額]。
3. 選取您的訂用帳戶，即可檢視配額限制。

## <a name="increase-a-batch-ai-cores-quota"></a>增加 Batch AI 的核心配額

請使用 [Azure 入口網站][portal]，遵循下列步驟來要求增加您 Batch AI 訂用帳戶的配額。 

1. 在左側窗格中按一下 [所有服務]。 然後搜尋 **Batch AI**，按一下以開啟服務。
2. 按一下 Batch AI 功能表中的 [新增支援要求]。
3. 在 [基本] 中：
   
    a. **問題類型** > **配額**
   
    b. [訂用帳戶] > 選取您的訂用帳戶。
   
    c. [配額類型] > [Batch AI]
   
    d. [支援方案] > 選取您的支援方案。

    按 [下一步] 。
4. 在 [問題]：
   
    a. 根據[商業影響][support_sev]選取 [嚴重性]。
   
    b. 在 [配額詳細資料] 中，指定位置、配額類型和資源類型。 指定您想要求的新限制。 按一下 [儲存並繼續]。

    c. 選擇性 - 上傳相關檔案，提供更多資訊說明您希望增加的原因。
   
    按 [下一步] 。
5. 在 [合約資訊] 中：
   
    a. 選取 [偏好的連絡方式]。
   
    b. 確認並輸入必要的連絡人詳細資料。
   
    按一下 [建立]  提交支援要求。

一旦您上傳支援要求，Azure 支援會與您連絡。 完成要求最多需要花費 2 個工作天。


## <a name="next-steps"></a>後續步驟

在熟悉配額限制後，請參閱下列文章開始使用 Batch AI。

> [!div class="nextstepaction"]
> [Batch AI 快速入門教學課程](quickstart-tensorflow-training-cli.md)
> [Batch AI 訣竅 (英文)](https://github.com/Azure/BatchAI/tree/master/recipes)
> [深入了解 Batch AI 資源](resource-concepts.md)

[portal]: https://portal.azure.com
[support_sev]: http://aka.ms/supportseverity