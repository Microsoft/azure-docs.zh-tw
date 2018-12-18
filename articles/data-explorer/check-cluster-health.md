---
title: 檢查 Azure 資料總管叢集的健康情況
description: 本文說明判斷「Azure 資料總管」叢集狀況是否良好的步驟。
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
services: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 0746247d2c912ba66e81b95f45b168e32b522130
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988422"
---
# <a name="check-the-health-of-an-azure-data-explorer-cluster"></a>檢查 Azure 資料總管叢集的健康情況

影響「Azure 資料總管」叢集健康情況的因素有數個，包括 CPU、記憶體及磁碟子系統。 本文說明一些您可採取來評估叢集健康情況的步驟。

1. 登入 [https://dataexplorer.azure.com](https://dataexplorer.azure.com)。

1. 在左窗格中，選取您的叢集，然後執行下列命令。

    ```Kusto
    .show diagnostics
    | project IsHealthy
    ```
    輸出 1 時表示狀況良好；輸出 0 時表示狀況不良。

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至您的叢集。

1. 在 [監視] 底下，選取 [計量]，然後選取 [保持運作]，如下圖所示。 輸出值如果接近 1，表示叢集狀況良好。

    ![叢集保持運作計量](media/check-cluster-health/portal-metrics.png)

1. 新增其他計量 (例如 CPU 和記憶體快取) 來評估叢集的資源使用率。

1. 如果您在診斷叢集健康情況問題方面需要協助，請在 [Azure 入口網站](https://portal.azure.com)中開啟支援要求。