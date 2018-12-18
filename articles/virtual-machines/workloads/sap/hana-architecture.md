---
title: SAP HANA on Azure (大型執行個體) 的架構 | Microsoft Docs
description: 如何部署 SAP HANA on Azure (大型執行個體) 的架構。
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
ms.date: 09/04/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7af578cf282c1bb8d8d7d00fee57bafed32b9a0e
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030543"
---
# <a name="sap-hana-large-instances-architecture-on-azure"></a>SAP HANA on Azure (大型執行個體) 架構

簡要來說，SAP HANA on Azure (大型執行個體) 解決方案會將 SAP 應用程式層放在 VM 中。 資料庫層則放在 SAP TDI 設定的硬體上，而此硬體是位於相同 Azure 區域中連線至 Azure IaaS 的「大型執行個體」戳記中。

> [!NOTE]
> 請在與 SAP DBMS 層相同的 Azure 區域中部署 SAP 應用程式層。 此規則詳載於已針對 Azure 上的 SAP 工作負載發佈的相關資訊中。 

SAP HANA on Azure (大型執行個體) 的整體架構會提供一個經 SAP TDI 認證的硬體組態 (適用於 SAP HANA 資料庫的非虛擬化、裸機、高效能伺服器)。 此外也提供 Azure 的能力與彈性來調整 SAP 應用程式層的資源，以符合您的需求。

![SAP HANA on Azure (大型執行個體) 的架構概觀](./media/hana-overview-architecture/image1-architecture.png)

顯示的架構分成三個部分：

- **右邊**：顯示一個在資料中心執行不同應用程式，讓使用者能夠存取 LOB 應用程式 (例如 SAP) 的內部部署基礎結構。 在理想的情況下，這個內部部署基礎結構會接著以 [ExpressRoute](https://azure.microsoft.com/services/expressroute/) 連線至 Azure。

- **中間**：顯示 Azure IaaS，在此案例中，會使用 VM 來裝載 SAP 或其他利用 SAP HANA 作為 DBMS 系統的應用程式。 以 VM 提供的記憶體運作的較小 HANA 執行個體會與其應用程式層一起部署在 VM 中。 如需虛擬機器的詳細資訊，請參閱[虛擬機器](https://azure.microsoft.com/services/virtual-machines/)。

   Azure 網路服務可用來將 SAP 系統及其他應用程式一起分組到虛擬網路中。 這些虛擬網路會連線至內部部署系統和 SAP HANA on Azure (大型執行個體)。

   對於可在 Azure 中執行的 SAP NetWeaver 應用程式和資料庫，請參閱 [SAP 支援附註 #1928533 - Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型](https://launchpad.support.sap.com/#/notes/1928533)。 如需關於如何在 Azure 上部署 SAP 解決方案的說明文件，請參閱：

  -  [在 Windows 虛擬機器上使用 SAP](../../virtual-machines-windows-sap-get-started.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
  -  [在 Azure 虛擬機器上使用 SAP 解決方案](get-started.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

- **左邊**：顯示「Azure 大型執行個體」戳記中經 SAP HANA TDI 認證的硬體。 「HANA 大型執行個體」單位會使用與從內部部署連線至 Azure 時的相同技術，連線至您訂用帳戶的虛擬網路。

「Azure 大型執行個體」戳記本身結合了下列元件：

- **運算**：以 Intel Xeon E7-8890v3 或 Intel Xeon E7-8890v4 處理器為基礎的伺服器，可提供必要的運算能力，且經過 SAP HANA 的認證。
- **網路**：將運算、儲存體及 LAN 元件互連的整合式高速網路網狀架構。
- **儲存體**：可透過整合式網路網狀架構存取的儲存體基礎結構。 根據已部署的特定 SAP HANA on Azure (大型執行個體) 組態提供的特定儲存體容量。 若要使用更多儲存體容量，則需每月額外付費。

在「大型執行個體」戳記的多租用戶基礎結構內，是將客戶部署成隔離的租用戶。 在租用戶的部署中，您必須在 Azure 註冊內為 Azure 訂用帳戶命名。 此 Azure 訂用帳戶將是 HANA 大型執行個體據以付費的訂用帳戶。 這些租用戶與 Azure 訂用帳戶具有一對一關係。 就網路而言，從分屬於不同 Azure 訂用帳戶的不同虛擬網路，存取在某個 Azure 區域的某個租用戶中部署的 HANA 大型執行個體單位，是可行的。 這些 Azure 訂用帳戶必須屬於相同的 Azure 註冊。 

和 VM 一樣，SAP HANA on Azure (大型執行個體) 會提供於多個 Azure 區域中。 若要提供災害復原功能，您可以選擇加入。 某個地理政治區域內不同的大型執行個體戳記會彼此互連。 例如，為了進行災害復原複寫，可以透過專用網路連結來連線美國西部和美國東部的 HANA 大型執行個體戳記。 

就像使用「Azure 虛擬機器」時可在不同的 VM 類型間做選擇一樣，您也可以從針對不同 SAP HANA 工作負載類型量身打造的不同「HANA 大型執行個體」SKU 中做選擇。 SAP 會根據 Intel 處理器世代，為不同的工作負載套用記憶體對處理器插槽的比例。 下表顯示提供的 SKU 類型。

您可以在 [HLI 可用的 SKU](hana-available-skus.md)中找到可用的 SKU。

**後續步驟**
- 請參閱 [SAP HANA (大型執行個體) 網路架構](hana-network-architecture.md)