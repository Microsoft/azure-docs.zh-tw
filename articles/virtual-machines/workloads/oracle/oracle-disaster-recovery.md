---
title: Azure 環境中的 Oracle 災害復原案例概觀 | Microsoft Docs
description: Azure 環境中 Oracle Database 12c 資料庫的災害復原案例
services: virtual-machines-linux
documentationcenter: virtual-machines
author: romitgirdhar
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/02/2018
ms.author: rogirdh
ms.openlocfilehash: 9f525e68502e32a3f9c7e7cebe6d45627f9077c3
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495004"
---
# <a name="disaster-recovery-for-an-oracle-database-12c-database-in-an-azure-environment"></a>Azure 環境中 Oracle Database 12c 資料庫的災害復原

## <a name="assumptions"></a>假設

- 您已了解 Oracle Data Guard 的設計和 Azure 環境。


## <a name="goals"></a>目標
- 設計符合您災害復原 (DR) 需求的拓撲和設定。

## <a name="scenario-1-primary-and-dr-sites-on-azure"></a>案例 1：Azure 上的主要和 DR 站台

客戶將 Oracle 資料庫設定在主要站台上。 DR 站台位於不同的區域中。 客戶使用 Oracle Data Guard 在這些站台間快速地進行復原。 主要站台另外還擁有次要資料庫以供進行報告和其他用途。 

### <a name="topology"></a>拓撲

以下是 Azure 設定的摘要：

- 兩個站台 (主要站台和 DR 站台)
- 兩個虛擬網路
- 兩個具有 Data Guard 的 Oracle 資料庫 (主要和待命)
- 兩個具有 Golden Gate 或 Data Guard 的 Oracle 資料庫 (僅限主要站台)
- 兩個應用程式服務，一個在主要站台上，一個在 DR 站台上
- 一個可用性設定組，用於主要站台上的資料庫和應用程式服務
- 每個站台上有一個 Jumpbox，其只能存取私人網路，並只允許系統管理員登入
- Jumpbox、應用程式服務、資料庫和 VPN 閘道位於不同的子網路
- 對應用程式和資料庫子網路強制執行 NSG

![DR 拓撲頁面的螢幕擷取畫面](./media/oracle-disaster-recovery/oracle_topology_01.png)

## <a name="scenario-2-primary-site-on-premises-and-dr-site-on-azure"></a>案例 2：內部部署環境的主要站台和 Azure 上的 DR 站台

客戶具有內部部署 Oracle 資料庫設定 (主要站台)。 DR 站台是在 Azure 上。 Oracle Data Guard 用於這些站台間的快速復原。 主要站台另外還擁有次要資料庫以供進行報告和其他用途。 

這項設定有兩種方法。

### <a name="approach-1-direct-connections-between-on-premises-and-azure-requiring-open-tcp-ports-on-the-firewall"></a>方法 1：讓內部部署環境與 Azure 直接連線，此方法需要在防火牆上開啟 TCP 連接埠 

不建議直接連線，因為這些連線會將 TCP 連接埠對外公開。

#### <a name="topology"></a>拓撲

以下是 Azure 設定的摘要：

- 一個 DR 站台 
- 一個虛擬網路
- 一個具有 Data Guard 的 Oracle 資料庫 (使用中)
- DR 站台上的一個應用程式服務
- 一個 Jumpbox，其只能存取私人網路，並只允許系統管理員登入
- Jumpbox、應用程式服務、資料庫和 VPN 閘道位於不同的子網路
- 對應用程式和資料庫子網路強制執行 NSG
- NSG 原則/規則，以允許輸入 TCP 連接埠 1521 (或使用者定義的連接埠)
- NSG 原則/規則，以便限制只有內部部署 IP 位址 (DB 或應用程式) 可以存取虛擬網路

![DR 拓撲頁面的螢幕擷取畫面](./media/oracle-disaster-recovery/oracle_topology_02.png)

### <a name="approach-2-site-to-site-vpn"></a>方法 2：站對站 VPN
站對站 VPN 是更好的方法。 如需設定 VPN 的詳細資訊，請參閱[使用 CLI 建立具有站對站 VPN 連線的虛擬網路](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli)。

#### <a name="topology"></a>拓撲

以下是 Azure 設定的摘要：

- 一個 DR 站台 
- 一個虛擬網路 
- 一個具有 Data Guard 的 Oracle 資料庫 (使用中)
- DR 站台上的一個應用程式服務
- 一個 Jumpbox，其只能存取私人網路，並只允許系統管理員登入
- Jumpbox、應用程式服務、資料庫和 VPN 閘道位於不同的子網路
- 對應用程式和資料庫子網路強制執行 NSG
- 內部部署與 Azure 之間的站對站 VPN 連線

![DR 拓撲頁面的螢幕擷取畫面](./media/oracle-disaster-recovery/oracle_topology_03.png)

## <a name="additional-reading"></a>其他閱讀資料

- [在 Azure 上設計和實作 Oracle 資料庫](oracle-design.md)
- [設定 Oracle Data Guard](configure-oracle-dataguard.md)
- [設定 Oracle Golden Gate](configure-oracle-golden-gate.md)
- [Oracle 備份和復原](oracle-backup-recovery.md)


## <a name="next-steps"></a>後續步驟

- [教學課程︰建立高可用性 VM](../../linux/create-cli-complete.md)
- [瀏覽 VM 部署 Azure CLI 範例](../../linux/cli-samples.md)
