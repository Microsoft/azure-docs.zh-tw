---
title: 使用 Azure 網路監看員功能所需的權限 | Microsoft Docs
description: 了解使用網路監看員功能時，需要哪些 Azure 角色型存取控制權限。
services: network-watcher
documentationcenter: ''
author: jimdial
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: network-watcher
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: jdial
ms.openlocfilehash: 7d0f0367a4126e7cecd34b39e6e5065e7d4fd90a
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287102"
---
# <a name="role-based-access-control-permissions-required-to-use-network-watcher-capabilities"></a>使用網路監看員功能時所需的角色型存取控制權限

Azure 角色型存取控制 (RBAC) 可將指派給組織成員的動作，限縮在該成員完成分配到的責任所需的範圍內。 若要使用網路監看員的功能，登入 Azure 的帳號必須受指派為[擁有者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#owner)、[參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#contributor)、[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#network-contributor)等內建角色，或者是受指派以下段落中各項網路監看員功能所屬動作的[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。 若要深入了解網路監看員的功能，請參閱[網路監看員是什麼？](network-watcher-monitoring-overview.md)。

## <a name="network-watcher"></a>網路監看員

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/read                              | 取得網路監看員                                          |
| Microsoft.Network/networkWatchers/write                             | 建立或更新網路監看員                             |
| Microsoft.Network/networkWatchers/delete                            | 刪除網路監看員                                       |

## <a name="nsg-flow-logs"></a>NSG 流量記錄

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/configureFlowLog/action           | 設定流量記錄                                           |
| Microsoft.Network/networkWatchers/queryFlowLogStatus/action         | 流量記錄的查詢狀態                                    |

## <a name="connection-troubleshoot"></a>針對連線問題進行疑難排解

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/queryTroubleshootResult/action    | 連線疑難排解測試的查詢結果                |
| Microsoft.Network/networkWatchers/troubleshoot/action               | 執行連線疑難排解測試                             |

## <a name="connection-monitor"></a>連線監視

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/connectionMonitors/start/action   | 啟動連線監視                                     |
| Microsoft.Network/networkWatchers/connectionMonitors/stop/action    | 停止連線監視                                      |
| Microsoft.Network/networkWatchers/connectionMonitors/query/action   | 查詢連線監視                                     |
| Microsoft.Network/networkWatchers/connectionMonitors/read           | 取得連線監視                                       |
| Microsoft.Network/networkWatchers/connectionMonitors/write          | 建立連線監視                                    |
| Microsoft.Network/networkWatchers/connectionMonitors/delete         | 刪除連線監視                                    |

## <a name="packet-capture"></a>封包擷取

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/packetCaptures/queryStatus/action | 查詢封包擷取的狀態                           |
| Microsoft.Network/networkWatchers/packetCaptures/stop/action        | 停止封包擷取                                          |
| Microsoft.Network/networkWatchers/packetCaptures/read               | 取得封包擷取                                           |
| Microsoft.Network/networkWatchers/packetCaptures/write              | 建立封包擷取                                        |
| Microsoft.Network/networkWatchers/packetCaptures/delete             | 刪除封包擷取                                        |

## <a name="ip-flow-verify"></a>IP 流量驗證

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/ipFlowVerify/action               | 驗證 IP 流量                                              |

## <a name="next-hop"></a>下一個躍點

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/nextHop/action                    | 從 VM 取得下一個躍點                                     |

## <a name="network-security-group-view"></a>網路安全性群組檢視

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/securityGroupView/action          | 檢視安全性群組                                           |

## <a name="topology"></a>拓撲

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/topology/action                   | 取得拓撲                                                   |

## <a name="reachability-report"></a>連線性報告

| 動作                                                              | 名稱                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/azureReachabilityReport/action    | 取得 Azure 連線性報告                               |

## <a name="additional-actions"></a>其他動作

網路監看員功能也需要下列動作：

- Microsoft.Storage/Read
- Microsoft.Authorization/Read
- Microsoft.Resources/subscriptions/resourceGroups/Read
- Microsoft.Storage/storageAccounts/listServiceSas/Action
- Microsoft.Storage/storageAccounts/listAccountSas/Action
- Microsoft.Storage/storageAccounts/listKeys/Action
- Microsoft.Compute/virtualMachines/Read
- Microsoft.Compute/virtualMachines/Write
- Microsoft.Compute/virtualMachineScaleSets/Read
- Microsoft.Compute/virtualMachineScaleSets/Write
- Microsoft.Insights/alertRules/*
- Microsoft.Support/*