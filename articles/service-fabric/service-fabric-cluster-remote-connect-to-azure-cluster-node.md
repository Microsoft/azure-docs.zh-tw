---
title: 從遠端連線 Azure Service Fabric 叢集節點 | Microsoft Docs
description: 了解如何從遠端連線擴展集執行個體 (Service Fabric 叢集節點)。
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: timlt
editor: ''
ms.assetid: 5441e7e0-d842-4398-b060-8c9d34b07c48
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/23/2018
ms.author: aljo
ms.openlocfilehash: 28424f9a7a0f77882ee3360c5599549303075c18
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34642568"
---
# <a name="remote-connect-to-a-virtual-machine-scale-set-instance-or-a-cluster-node"></a>遠端連線到虛擬機器擴展集執行個體或叢集節點
在 Azure 中執行的 Service Fabric 叢集內，您定義的每個叢集節點類型都會[設定虛擬機器的個別擴展](service-fabric-cluster-nodetypes.md)。  您可以從遠端連線特定的擴展集執行個體 (叢集節點)。  不同於單一執行個體的 VM，VM 擴展集的執行個體不會有自己的虛擬 IP 位址。 當您要尋找可用來遠端連線至特定執行個體的 IP 位址和連接埠時，可能有點困難。

若要找出可用來遠端連線至特定執行個體的 IP 位址和連接埠，請完成下列步驟。

1. 取得遠端桌面通訊協定 (RDP) 的輸入 NAT 規則。

    在叢集中定義的每個節點類型通常都有它自己的虛擬 IP 位址和專用負載平衡器。 根據預設，節點類型的負載平衡器名稱採用下列格式：LB-{cluster-name}-{node-type}；例如，LB-mycluster-FrontEnd。 
    
    在 Azure 入口網站中您負載平衡器的頁面上，選取 [設定] > [輸入 NAT 規則]： 

    ![負載平衡器輸入 NAT 規則](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/lb-window.png)

    下列螢幕擷取畫面顯示 FrontEnd 節點類型的輸入 NAT 規則： 

    ![負載平衡器輸入 NAT 規則](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/nat-rules.png)

    針對每個節點，IP 位址會出現在 [目的地]資料行中，[目標] 資料行會提供擴展集執行個體，而 [服務] 資料行會提供連接埠號碼。 針對遠端連線，連接埠會以遞增順序配置給每個節點 (開頭為連接埠 3389)。

    您也可以針對叢集，在 Resource Manager 範本的 `Microsoft.Network/loadBalancers` 區段中找到輸入 NAT 規則。
    
2. 若要確認節點的輸入連接埠與目標連接埠對應，您可以按一下其規則並查看 [目標連接埠] 值。 下列螢幕擷取畫面顯示前一個步驟中 **FrontEnd (執行個體 1)** 節點的輸入 NAT 規則。 請注意，雖然 (輸入) 連接埠號碼是 3390，但目標連接埠會對應至連接埠 3389 (目標上 RDP 服務的連接埠)。  

    ![目標連接埠對應](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/port-mapping.png)

    根據預設，若為 Windows 叢集，目標連接埠為連接埠 3389，它會對應到目標節點上的 RDP 服務。 若為 Linux 叢集，目標連接埠為連接埠 22，它會對應到安全殼層 (SSH) 服務。

3. 遠端連線到特定節點 (擴展集執行個體)。 您可以使用當您建立叢集時所設定的使用者名稱和密碼，或您已設定的任何其他認證。 

    下列螢幕擷取畫面顯示如何使用「遠端桌面連線」來連線到 Windows 叢集中的 **FrontEnd (執行個體 1)**：
    
    ![遠端桌面連線](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/rdp-connect.png)

    在 Linux 節點上，您可以透過 SSH 連線 (為求簡單明瞭，下列範例會重複使用相同的 IP 位址和連接埠)：

    ``` bash
    ssh SomeUser@40.117.156.199 -p 3390
    ```


若要了解後續步驟，請閱讀下列文章：
* 請參閱[「到處部署」功能和與 Azure 受控叢集比較的概觀](service-fabric-deploy-anywhere.md)。
* 了解[叢集安全性](service-fabric-cluster-security.md)。
* 部署後在叢集 VM 上[更新 RDP 連接埠範圍值](./scripts/service-fabric-powershell-change-rdp-port-range.md)
* 變更叢集 VM 的[管理員使用者名稱和密碼](./scripts/service-fabric-powershell-change-rdp-user-and-pw.md)

