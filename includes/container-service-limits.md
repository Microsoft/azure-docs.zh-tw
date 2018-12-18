---
title: 包含檔案
description: 包含檔案
services: container-service
author: dlepow
ms.service: container-service
ms.topic: include
ms.date: 10/11/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 4251f379c517d5ccfd0430987e3d5280208590ff
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49400158"
---
| 資源 | 預設限制 |
| --- | :--- |
| 每個訂用帳戶的最大叢集數 | 100 |
| 每個叢集的最大節點數 | 100 |
| 每個節點的最大 Pod 數：採用 Kubenet 的[基本網路功能][basic-networking] | 110 |
| 每個節點的最大 Pod 數：採用 Azure CNI 的[進階網路功能][advanced-networking] | Azure CLI 部署：30<sup>1</sup><br />Resource Manager 範本：30<sup>1</sup><br />入口網站部署：30 |

<sup>1</sup> 當您使用 Azure CLI 或 Resource Manager 範本部署 AKS 叢集時，此值最高可設定為**每個節點 110 個 Pod**。 在您已部署 AKS 叢集之後，或如果您使用 Azure 入口網站部署叢集，就無法設定每個節點的 Pod 上限。<br />

<!-- LINKS - Internal -->
[basic-networking]: ../articles/aks/concepts-network.md#basic-networking
[advanced-networking]: ../articles/aks/concepts-network.md#advanced-networking

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
