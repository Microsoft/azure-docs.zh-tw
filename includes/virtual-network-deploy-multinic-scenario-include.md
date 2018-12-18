---
title: 包含檔案
description: 包含檔案
services: virtual-network
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 04/13/2018
ms.author: genli
ms.custom: include file
ms.openlocfilehash: cab04a7eafbc21e0d26cd5a287f3dbee8d3d22b7
ms.sourcegitcommit: 150a40d8ba2beaf9e22b6feff414f8298a8ef868
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "31805093"
---
## <a name="scenario"></a>案例
本文件會逐步說明一個在 VM 中使用多個 NIC 的部署特殊案例。 在此案例中， Azure 中託管了兩層式 IaaS 工作負載。 在虛擬網路 (VNet) 中每一層皆部署在各自的子網路。 前端層是由數個 Web 伺服器組成，在負載平衡器以群組方式結合，以達高可用性。 後端層是由數個資料庫伺服器所組成。 這些資料庫伺服器各自會部屬兩個 NIC，一個用資料庫存取，另一個用於管理。 此案例也包括了網路安全群組 (NSG)，用來控制每個子網路可允許哪些流量通過，以及部署中的 NIC 。 下圖顯示此案例的基本架構：

![多個 NIC 案例](./media/virtual-network-deploy-multinic-scenario-include/Figure1.png)

