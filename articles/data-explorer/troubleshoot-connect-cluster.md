---
title: 無法在 Azure 資料總管中連線至叢集
description: 本文描述在 Azure 資料總管中連線至叢集的疑難排解步驟。
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
services: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: f66dcc55b01b407c59c65ea300757ab4ee1002ac
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965053"
---
# <a name="troubleshoot-failure-to-connect-to-a-cluster-in-azure-data-explorer"></a>移難排解：無法在 Azure 資料總管中連線至叢集

如果您無法在 Azure 資料總管中連線至叢集，請遵循下列步驟。

1. 請確認連接字串正確無誤。 格式應為：`https://<ClusterName>.<Region>.kusto.windows.net`，如下列範例：`https://docscluster.westus.kusto.windows.net`。

1. 確定您有足夠的權限。 否則，您將會得到*未獲授權*的回應。

    如需有關權限的詳細資訊，請參閱[管理資料庫權限](manage-database-permissions.md)。 如有必要，請與叢集系統管理員合作，讓他們將您新增到適當的角色。

1. 請確認叢集未遭到刪除：檢閱您訂用帳戶中的活動記錄檔。

1. 查看 [Azure 服務健康情況儀表板](https://azure.microsoft.com/status/>)。 請在您要嘗試連線至叢集的區域中尋找 Azure 資料總管的狀態。

    如果狀態不是**良好** (綠色核取記號)，請在狀態改善之後嘗試連線至叢集。

1. 若您仍然需要協助來解決問題，請在 [Azure 入口網站](https://portal.azure.com)中開啟支援要求。