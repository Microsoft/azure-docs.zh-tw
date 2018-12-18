---
title: Azure Service Fabric CLI- sfctl | Microsoft Docs
description: 描述 Service Fabric CLI sfctl 命令。
services: service-fabric
documentationcenter: na
author: Christina-Kang
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: cli
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 07/31/2018
ms.author: bikang
ms.openlocfilehash: b6e3c769307f037c9f31dbb3f1a86675145ec1f6
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495402"
---
# <a name="sfctl"></a>sfctl
用於管理 Service Fabric 叢集和實體的命令。 這個版本與 Service Fabric 6.3 執行階段相容。

命令會依循「名詞-動詞」模式。 如需詳細資訊，請參閱＜子群組＞。

## <a name="subgroups"></a>子群組
|子群組|說明|
| --- | --- |
| [application](service-fabric-sfctl-application.md) | 建立、刪除與管理應用程式和應用程式類型。 |
| [chaos](service-fabric-sfctl-chaos.md) | 啟動、停止及報告 chaos 測試服務。 |
| [cluster](service-fabric-sfctl-cluster.md) | 選取、管理和操作 Service Fabric 叢集。 |
| [compose](service-fabric-sfctl-compose.md) | 建立、刪除和管理 Docker Compose 應用程式。 |
| [container](service-fabric-sfctl-container.md) | 在叢集節點上執行容器相關命令。 |
| [is](service-fabric-sfctl-is.md) | 查詢命令，並將其傳送至基礎結構服務。 |
| [node](service-fabric-sfctl-node.md) | 管理形成叢集的節點。 |
| [partition](service-fabric-sfctl-partition.md) | 查詢和管理任何服務的資料分割。 |
| [property](service-fabric-sfctl-property.md) | 在 Service Fabric 名稱底下儲存和查詢屬性。 |
| [replica](service-fabric-sfctl-replica.md) | 管理屬於服務分割區的複本。 |
| [rpm](service-fabric-sfctl-rpm.md) | 查詢命令，並將其傳送至修復管理員服務。 |
| [sa-cluster](service-fabric-sfctl-sa-cluster.md) | 管理獨立 Service Fabric 叢集。 |
| [service](service-fabric-sfctl-service.md) | 建立、刪除與管理服務、服務類型和服務套件。 |
| [store](service-fabric-sfctl-store.md) | 在叢集映像存放區上執行基本檔案層級作業。 |

## <a name="next-steps"></a>後續步驟
- [設定](service-fabric-cli.md) Service Fabric CLI。
- 了解如何使用[範例指令碼](/azure/service-fabric/scripts/sfctl-upgrade-application)來使用 Service Fabric CLI。