---
title: 從 VNET 連線到 Azure Data Lake Storage Gen1 | Microsoft Docs
description: 從 Azure VNET 連線到 Azure Data Lake Storage Gen1
services: data-lake-store,data-catalog
documentationcenter: ''
author: esung22
manager: jhubbard
editor: cgronlun
ms.assetid: 683fcfdc-cf93-46c3-b2d2-5cb79f5e9ea5
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 01/31/2018
ms.author: elsung
ms.openlocfilehash: 130d0154fc0558ae7284e8407ba88fda3a2a53d5
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391295"
---
# <a name="access-azure-data-lake-storage-gen1-from-vms-within-an-azure-vnet"></a>從 Azure VNET 內的虛擬機器存取 Azure Data Lake Storage Gen1
Azure Data Lake Storage Gen1 是公用網際網路 IP 位址上執行的 PaaS 服務。 可以連線到公用網際網路的任何伺服器，通常也可以連線到 Azure Data Lake Storage Gen1 端點。 根據預設，Azure VNET 中的所有虛擬機器皆可存取網際網路，因此，可以存取 Azure Data Lake Storage Gen1。 不過，也可以將 VNET 中的 VM 設定為無法存取網際網路。 對於這類虛擬機器，也會限制對 Azure Data Lake Storage Gen1 的存取。 封鎖 Azure VNET 中 VM 的公用網際網路存取，可以使用下列任一個方法來完成：

* 藉由設定網路安全性群組 (NSG)
* 藉由設定使用者定義路由 (UDR)
* 藉由在使用 ExpressRoute 時，透過可封鎖網際網路存取的 BGP (產業標準動態路由通訊協定) 來交換路由

在本文中，您將了解如何從 Azure VM 啟用對 Azure Data Lake Storage Gen1 的存取，Azure VM 僅限於使用前述的三種方法之一存取資源。

## <a name="enabling-connectivity-to-azure-data-lake-storage-gen1-from-vms-with-restricted-connectivity"></a>從具有受限制連線的 VM 啟用對 Azure Data Lake Storage Gen1 的連線
若要從這類虛擬機器存取 Azure Data Lake Storage Gen1，您必須設定這些虛擬機器，以存取 Azure Data Lake Storage Gen1 帳戶可用的 IP 位址。 您可以透過解析您帳戶 (`<account>.azuredatalakestore.net`) 的 DNS 名稱來識別 Data Lake Storage Gen1 帳戶的 IP 位址。 若要解析帳戶的 DNS 名稱，您可以使用諸如 **nslookup** 的工具。 在您的電腦上開啟命令提示字元，並執行下列命令：

    nslookup mydatastore.azuredatalakestore.net

輸出結果類似下面。 對應 **Address** 屬性的值是與您的 Data Lake Storage Gen1 帳戶相關聯的 IP 位址。

    Non-authoritative answer:
    Name:    1434ceb1-3a4b-4bc0-9c69-a0823fd69bba-mydatastore.projectcabostore.net
    Address:  104.44.88.112
    Aliases:  mydatastore.azuredatalakestore.net


### <a name="enabling-connectivity-from-vms-restricted-by-using-nsg"></a>使用 NSG 從受限制的 VM 啟用連線
使用 NSG 規則來封鎖對網際網路的存取時，您接著可以建立允許存取 Data Lake Storage Gen1 IP 位址的另一個 NSG。 如需有關 NSG 規則的詳細資訊，請參閱[網路安全性群組概觀](../virtual-network/security-overview.md)。 如需如何建立 NSG 的指示，請參閱[如何建立網路安全性群組](../virtual-network/tutorial-filter-network-traffic.md)。

### <a name="enabling-connectivity-from-vms-restricted-by-using-udr-or-expressroute"></a>使用 UDR 或 ExpressRoute 從受限制的 VM 啟用連線
路由時會使用 UDR 或 BGP 交換路由來封鎖對網際網路的存取，必須設定特殊的路由，才能使這類子網路中的 VM 可以存取 Data Lake Storage Gen1 端點。 如需詳細資訊，請參閱[使用者定義路由概觀](../virtual-network/virtual-networks-udr-overview.md)。 如需建立 UDR 的指示，請參閱[在 Resource Manager 中建立 UDR](../virtual-network/tutorial-create-route-table-powershell.md)。

### <a name="enabling-connectivity-from-vms-restricted-by-using-expressroute"></a>使用 ExpressRoute 從受限制的 VM 啟用連線
設定 ExpressRoute 電路時，內部部署伺服器可以透過公用對等互連來存取 Data Lake Storage Gen1。 如需針對公用互連設定 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](../expressroute/expressroute-faqs.md)。

## <a name="see-also"></a>另請參閱
* [Azure Data Lake Storage Gen1 概觀](data-lake-store-overview.md)
* [保護儲存在 Azure Data Lake Storage Gen1 中的資料](data-lake-store-security-overview.md)

