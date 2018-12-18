---
title: Azure Container Instances 配額和區域可用性
description: Azure Container Instances 服務的預設配額和區域可用性。
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: overview
ms.date: 02/27/2018
ms.author: danlep
ms.openlocfilehash: 427dd8bd4abb72e2750752d828e189921401e9e0
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48902345"
---
# <a name="quotas-and-region-availability-for-azure-container-instances"></a>Azure Container Instances 的配額和區域可用性

所有 Azure 服務都包含資源和功能的特定預設限制和配額。 下列各節詳細說明數個 Azure Container Instances (ACI) 資源的預設資源限制，以及 Azure 區域中 ACI 服務的可用性。

## <a name="service-quotas-and-limits"></a>服務配額和限制

[!INCLUDE [container-instances-limits](../../includes/container-instances-limits.md)]

## <a name="region-availability"></a>區域可用性

具有指定 CPU 和記憶體限制的 Azure Container Instances 可於下列區域取得。

| 位置 | 作業系統 | CPU | 記憶體 (GB) |
| -------- | -- | :---: | :-----------: |
| 美國東部、北歐、西歐、美國西部、美國西部 2 | Linux | 4 | 14 |
| 澳大利亞東部、美國東部 2、東南亞 | Linux | 2 | 7 |
| 印度中部、美國中南部 | Linux | 2 | 3.5 |
| 美國東部、西歐、美國西部 | Windows | 4 | 14 |
| 澳大利亞東部、印度中部、美國東部 2、北歐、美國中南部、東南亞、美國西部 2 | Windows | 2 | 3.5 |

在這些資源限制下建立的容器執行個體都會受限於部署地區的可用性。 地區負載過重時，您在部署執行個體時可能會失敗。 若要減少這類的部署失敗，請嘗試部署 CPU 和記憶體設定較低階的執行個體，或過一段時間再部署。

在 [aka.ms/aci/feedback](https://aka.ms/aci/feedback) 讓小組知道其他必要的區域或增加的 CPU/記憶體限制。

如需有關為容器執行個體部署疑難排解的詳細資訊，請參閱[為 Azure Container Instances 的部署問題疑難排解](container-instances-troubleshooting.md)。

## <a name="next-steps"></a>後續步驟

某些預設限制和配額可以增加。 若要要求增加一或多個支援此類增加的資源，請提交 [Azure 支援要求][azure-support] (針對 [問題類型] 選取 [配額])。

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
