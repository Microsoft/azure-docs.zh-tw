---
title: 從虛擬機器到 SAP HANA on Azure (大型執行個體) 的連線設定 | Microsoft Docs
description: 從虛擬機器到 SAP HANA on Azure (大型執行個體) 的連線設定。
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: f0e09e6dfce5d0ede525f461193866219b7f9429
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44167621"
---
# <a name="connecting-azure-vms-to-hana-large-instances"></a>將 Azure VM 連接到 HANA 大型執行個體

如 [Azure 上 SAP HANA (大型執行個體) 的概觀和架構](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)中所述，Azure 中搭配 SAP 應用程式層的「HANA 大型執行個體」最基本部署看起來如下：

![連接到 SAP HANA on Azure (大型執行個體) 與內部部署環境的 Azure VNet](./media/hana-overview-architecture/image3-on-premises-infrastructure.png)

更仔細審視 Azure VNet 端之後，我們發現有下列需求：

- Azure VNet 的定義，SAP 應用程式層 VM 會部署在此。
- 這自動意謂著已在 Azure Vnet 中定義真正用來作為 VM 部署目的地的預設子網路。
- 所建立的 Azure VNet 必須至少有一個 VM 子網路和一個「ExpressRoute 閘道」子網路。 您應該依下列各節所指定及討論的，將 IP 位址範圍指派給這些子網路。

因此，讓我們更進一步了解如何為「HANA 大型執行個體」建立 Azure VNet

## <a name="creating-the-azure-vnet-for-hana-large-instances"></a>為 HANA 大型執行個體建立 Azure VNet

>[!Note]
>您必須使用 Azure Resource Manager 部署模型來為「HANA 大型執行個體」建立 Azure VNet。 HANA 大型執行個體解決方案不支援舊版的 Azure 部署模型 (通常稱為傳統部署模型)。

您可以使用 Azure 入口網站、PowerShell、Azure 範本或 Azure CLI 來建立 VNet (請參閱[使用 Azure 入口網站建立虛擬網路](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network))。 在下例中，我們將探討透過 Azure 入口網站建立的 VNet。

如果要透過 Azure 入口網站來探討 Azure VNet 的定義，讓我們來研究一下其中的某些定義，以及這些定義與我們列出的不同 IP 位址範圍如何關聯。 當我們提到**位址空間**時，指的是 Azure VNet 獲允許使用的位址空間。 此位址空間也是 VNet 用來傳播 BGP 路由的位址範圍。 您可以在下列位置看到此「位址空間」：

![Azure 入口網站中顯示的 Azure VNet 位址空間](./media/hana-overview-connectivity/image1-azure-vnet-address-space.png)

在前例中，使用了 10.16.0.0/16 來賦予 Azure VNet 更大、更廣的可用 IP 位址範圍。 這意謂著此 VNet 內後續子網路之所有 IP 位址範圍的範圍都可在該「位址空間」內。 通常針對 Azure 中的單一 VNet，我們並不建議使用這麼大的位址範圍。 但為了進一步了解，讓我們來研究一下 Azure VNet 中定義的子網路：

![Azure VNet 子網路及其 IP 位址範圍](./media/hana-overview-connectivity/image2b-vnet-subnets.png)

如您所見，我們看到的是含有第一個 VM 子網路 (這裡稱為 'default') 和稱為 'GatewaySubnet' 之子網路的 VNet。
在下節中，我們將圖中名為 'default' 之子網路的 IP 位址範圍稱為 **Azure VM 子網路 IP 位址範圍**。 在接下來的小節中，我們將「閘道子網路」(Gateway Subnet) 的 IP 位址範圍稱為「VNet 閘道子網路 IP 位址範圍」。 

在上述兩張圖所示範的案例中，您會看到「VNet 位址空間」同時涵蓋「Azure VM 子網路 IP 位址範圍」和「VNet 閘道子網路 IP 位址範圍」。 

在您需要節約使用或有明確 IP 位址範圍的其他案例中，您可以將 VNet 的 **VNet 位址空間**限制成每個子網路所使用的特定範圍。 在此情況下，您可以將 VNet 的「VNet 位址空間」定義成多個特定範圍，如以下所示：

![含有兩個空間的「Azure VNet 位址空間」](./media/hana-overview-connectivity/image3-azure-vnet-address-space_alternate.png)

在本例中，**VNet 位址空間**定義了兩個空間。 這兩個空間與為 **Azure VM 子網路 IP 位址範圍**和 **VNet 閘道子網路 IP 位址範圍**定義的 IP 位址範圍相同。

您可以將您喜歡的任何命名標準用於這些租用戶子網路 (VM 子網路)。 不過，**每個 VNet 都必須要有、且只有一個閘道子網路**連接至 Azure (大型執行個體) ExpressRoute 線路上的 SAP HANA。 而且，**這個閘道子網路必須一律名為 "GatewaySubnet"**，以確保正確放置 ExpressRoute 閘道。

> [!WARNING] 
> 將閘道子網路一律命名為 "GatewaySubnet" 非常重要。

您可以使用多個 VM 子網路，甚至是使用非連續的位址範圍。 但如先前所述，VNet 的「VNet 位址空間」必須涵蓋這些位址範圍，不論是以彙總形式涵蓋，還是以 VM 子網路與閘道子網路的精確範圍清單涵蓋。

總結連接到「HANA 大型執行個體」之 Azure VNet 的重要事實：

- 執行 HANA 大型執行個體初始部署時，您需要向 Microsoft 提交 **VNet 位址空間**。 
- 「VNet 位址空間」可以是一個涵蓋「Azure VM 子網路 IP 位址範圍」和「VNet 閘道子網路 IP 位址範圍」之範圍的較大範圍。
- 或者，您可以用「VNet 位址空間」多重範圍的形式提交，這些範圍涵蓋「VM 子網路 IP 位址範圍」和「VNet 閘道子網路 IP 位址範圍」的不同 IP 位址範圍。
- 已定義的「VNet 位址空間」會用來傳播 BGP 路由。
- 閘道子網路的名稱必須是：**"GatewaySubnet"**。
- 在「HANA 大型執行個體」端會使用「VNet 位址空間」作為篩選條件，以允許或拒絕從 Azure 到「HANA 大型執行個體」單位的流量。 如果 Azure VNet 的 BGP 路由資訊與針對在 HANA 大型執行個體端上進行篩選而設定的 IP 位址範圍不符，就會發生連線問題。
- ＜將 VNet 連接到 HANA 大型執行個體 ExpressRoute＞一節將進一步討論閘道子網路的一些相關詳細資料



## <a name="different-ip-address-ranges-to-be-defined"></a>要定義的不同 IP 位址範圍 

前幾節中已介紹一些部署 HANA 大型執行個體所需的 IP 位址範圍。 但仍有一些其他重要的 IP 位址。 讓我們來瀏覽一些進一步的詳細資料。 傳送初始部署要求之前，必須定義下列 IP 位址，這些 IP 位址並非全都必須提交給 Microsoft：

- **VNet 位址空間：** 如前文所介紹，這是您在連接到 SAP HANA 大型執行個體環境的 Azure 虛擬網路 (VNet) 中，已指派 (或打算指派) 給位址空間參數的 IP 位址範圍。 建議此位址空間參數採用由 Azure VM 子網路範圍和 Azure 閘道子網路範圍所組成的多行值，如前文圖中所示。 此範圍「不得」與您的內部部署、伺服器 IP 集區或 ER-P2P 位址範圍重疊。 如何取得 IP 位址範圍？ 您的公司網路小組或服務提供者應該提供一或多個網路內未使用的 IP 位址範圍。 範例：如果您的 Azure VM 子網路 (參見前文) 是 10.0.1.0/24，而 Azure 閘道子網路 (參見後方) 是 10.0.2.0/28，則建議 Azure VNet 位址空間採用兩行，即 10.0.1.0/24 和 10.0.2.0/28。 雖然位址空間值可以彙總，但建議讓它們符合子網路範圍，以避免日後在網路中的其他地方，不小心重複使用在較大位址範圍內未使用的 IP 位址範圍。 **VNET 位址範圍是要求初始部署時，所必須提交給 Microsoft 的 IP 位址範圍**

- **Azure VM 子網路 IP 位址空間：** 如前文所述，這是您在連接到 SAP HANA 大型執行個體環境的 Azure VNET 中，已指派 (或打算指派) 給 Azure VNet 子網路參數的 IP 位址範圍。 此 IP 位址範圍可用來將 IP 位址指派給您的 Azure VM。 超出此範圍的 IP 位址可以連接到您的 SAP HANA 大型執行個體伺服器。 如果需要，可以使用多個 Azure VM 子網路。 Microsoft 建議為每個「Azure VM 子網路」指定 /24 CIDR 區塊。 此位址範圍必須是「Azure VNet 位址空間」中所使用值的一部分。 如何取得此 IP 位址範圍？ 您的公司網路小組或服務提供者應該提供一個目前網路內未使用的 IP 位址範圍。

- **VNet 閘道子網路 IP 位址範圍︰** 根據您打算使用的功能，建議的大小為︰
   - 強效 ExpressRoute 閘道：/26 位址區塊 - 類型 II 類別的 SKU
   - 與使用高效能 ExpressRoute 閘道 (或更小) 的 VPN 和 ExpressRoute 共存：/27 位址區塊
   - 所有其他情況：/28 位址區塊。 此位址範圍必須是「VNet 位址空間」值中所使用之值的一部分。 此位址範圍必須是您需要提交給 Microsoft 之「Azure VNet 位址空間」值中所使用值的一部分。 如何取得此 IP 位址範圍？ 您的公司網路小組或服務提供者應該提供一個目前網路內未使用的 IP 位址範圍。 

- **ER-P2P 連線的位址範圍：** 這是您 SAP HANA 大型執行個體 ExpressRoute (ER) P2P 連線的 IP 範圍。 此 IP 位址範圍必須是 /29 CIDR IP 位址範圍。 此範圍「不得」與您的內部部署或其他 Azure IP 位址範圍重疊。 此 IP 位址可用來設定從 Azure VNet ExpressRoute 閘道到 SAP HANA 大型執行個體伺服器的 ER 連線。 如何取得此 IP 位址範圍？ 您的公司網路小組或服務提供者應該提供一個目前網路內未使用的 IP 位址範圍。 **此範圍是要求初始部署時，需要提交給 Microsoft 的 IP 位址範圍**
  
- **伺服器 IP 集區位址範圍︰** 此 IP 位址範圍可用來將個別 IP 位址指派給 HANA 大型執行個體伺服器。 建議的子網路大小是 /24 CIDR 區塊 - 但如果需要，它也可以小到最小僅提供 64 個 IP 位址。 在這個範圍中，前 30 個 IP 位址要保留供 Microsoft 使用。 選擇範圍的大小時，請確定已將此實況納入考量。 此範圍「不得」與您的內部部署或其他 Azure IP 位址重疊。 如何取得此 IP 位址範圍？ 您的公司網路小組或服務提供者應該提供一個目前網路內未使用的「IP 位址範圍」。 要用於指派 SAP HANA on Azure (大型執行個體) 所需之特定 IP 位址的 /24 (建議使用) 唯一 CIDR 區塊。 **此範圍是要求初始部署時，需要提交給 Microsoft 的 IP 位址範圍**
 
雖然您需要定義和規劃上述的 IP 位址範圍，但並非全都需要傳送給 Microsoft。 總結以上所述，您必須列舉給 Microsoft 的 IP 位址範圍包括：

- Azure VNet 位址空間
- 用於 ER-P2P 連線的位址範圍
- 伺服器 IP 集區位址範圍

若要新增其他需要連接到「HANA 大型執行個體」的 VNet，您必須將新的「Azure VNet 位址空間」提交給 Microsoft。 

以下是不同範圍的範例，以及一些您需要設定並最後提供給 Microsoft 的範例範圍。 如您所見，在第一個範例中，Azure VNet 位址空間的值並未經過彙總，而是從第一個 Azure VM 子網路 IP位址範圍和 VNet 閘道子網路 IP 位址範圍的範圍定義而來。 在 Azure VNet 內使用多個 VM 子網路時，會根據隨「Azure VNet 位址空間」一併設定並提交之額外 VM 子網路的額外 IP 位址範圍運作。

![SAP HANA on Azure (大型執行個體) 最基本部署中所需的 IP 位址範圍](./media/hana-overview-connectivity/image4b-ip-addres-ranges-necessary.png)

您也可以彙總要提交給 Microsoft 的資料。 在該情況下，Azure VNet 的位址空間只會包含一個空間。 使用前例中所使用的 IP 位址範圍。 此彙總的 VNet 位址空間可能看起來像這樣：

![SAP HANA on Azure (大型執行個體) 最基本部署中所需 IP 位址範圍的第二個可能性](./media/hana-overview-connectivity/image5b-ip-addres-ranges-necessary-one-value.png)

如您在上面所見，我們並未採用兩個定義 Azure VNet 位址空間的較小範圍，而是採用一個涵蓋 4096 個 IP 位址範圍的較大範圍。 定義這樣大的「位址空間」會使得相當大的一些範圍沒有被使用。 由於「VNet 位址空間」值會用來傳播 BGP 路由，因此使用內部部署環境或您網路中其他地方未使用的範圍會造成路由問題。 所以，建議讓「位址空間」與所使用的實際子網路位址空間保持完全一致。 您一律可以在稍後視需要新增新的位址空間值，這不會導致 VNet 停機。
 
> [!IMPORTANT] 
> ER-P2P、伺服器 IP 集區、Azure VNet 位址空間的每個 IP 位址都「不得」彼此重疊或與網路上其他地方使用的任何其他範圍重疊；每個 IP 必須是個別的，並且如前兩圖所示，不是任何其他範圍的子網路。 如果兩個範圍之間發生重疊，Azure VNet 可能無法連接到 ExpressRoute 線路。

## <a name="next-steps-after-address-ranges-have-been-defined"></a>定義位址範圍之後的後續步驟
定義 IP 位址範圍之後，需要進行下列活動：

1. 將用於「Azure VNet 位址空間」、ER-P2P 連線及「伺服器 IP 集區位址範圍」的 IP 位址範圍與本文開頭所列其他資料一起提交。 此時，您也可以開始建立 VNet 和 VM 子網路。 
2. 由 Microsoft 在您的 Azure 訂用帳戶與「HANA 大型執行個體」戳記之間建立 ExpressRoute 線路。
3. 由 Microsoft 在「大型執行個體」戳記上建立租用戶網路。
4. Microsoft 會設定 SAP HANA on Azure (大型執行個體) 基礎結構中的網路功能，以接受來自您 Azure VNet 位址空間之將與 HANA 大型執行個體通訊的 IP 位址。
5. 依據所購買的特定 SAP HANA on Azure (大型執行個體) SKU，Microsoft 會在租用戶網路中指派一個計算單位、配置並掛接儲存體，以及安裝作業系統 (SUSE 或 Red Hat Linux)。 這些單位的 IP 位址是取自您提交給 Microsoft 的「伺服器 IP 集區位址範圍」。

在部署程序結束時，Microsoft 會將下列資料傳送給您：
- 將您的 Azure VNet 連接到 ExpressRoute 線路 (此線路會將 Azure VNet 連接到「HANA 大型執行個體」) 所需的資訊：
     - 授權金鑰
     - ExpressRoute PeerID
- 在您建立 ExpressRoute 線路和 Azure VNet 之後，用以存取「HANA 大型執行個體」的資料。

您也可以在 [SAP HANA 大型執行個體的端對端安裝](https://azure.microsoft.com/resources/sap-hana-on-azure-large-instances-setup/)一文中，找到連接 HANA 大型執行個體的序列。 該文件的範例部署會示範下列許多步驟。 

**後續步驟**

- 請參閱[將 VNet 連線到 HANA 大型執行個體 ExpressRoute](hana-connect-vnet-express-route.md)。