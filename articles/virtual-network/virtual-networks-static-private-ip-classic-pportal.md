---
title: 設定 VM (傳統) 的私人 IP 位址 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用 Azure 入口網站設定虛擬機器 (傳統) 的私人 IP 位址。
services: virtual-network
documentationcenter: na
author: genlin
manager: dcscontentpm
tags: azure-service-management
ms.assetid: b8ef8367-58b2-42df-9f26-3269980950b8
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2016
ms.author: genli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 6986f6f16cbd32d44223bba4f4be4577fa11258c
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98222899"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-classic-using-the-azure-portal"></a>使用 Azure 入口網站設定虛擬機器 (傳統) 的私人 IP 位址

[!INCLUDE [virtual-networks-static-private-ip-selectors-classic-include](../../includes/virtual-networks-static-private-ip-selectors-classic-include.md)]

[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文涵蓋之內容包括傳統部署模型。 您也可以 [管理資源管理員部署模型中的靜態私人 IP 位址](virtual-networks-static-private-ip-arm-pportal.md)。

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

接下來的範例步驟會預期已建立簡單的環境。 如果您想要執行如本文件中所顯示的步驟，請先建置 [建立 vnet](/previous-versions/azure/virtual-network/virtual-networks-create-vnet-classic-pportal)中所說明的測試環境。

## <a name="how-to-specify-a-static-private-ip-address-when-creating-a-vm"></a>建立 VM 時如何指定靜態私人 IP 位址
若要在名為 *TestVNet* 之 VNet 的 *FrontEnd* 子網路中建立名為 *DNS01* 的 VM，且其靜態私人 IP 為 *192.168.1.101*，請完成下列步驟：

1. 透過瀏覽器瀏覽至 https://portal.azure.com，並視需要使用您的 Azure 帳戶登入。
2. 選取 [**新增**  >  **計算**  >  **Windows Server 2012 R2 Datacenter**]，請注意 [**選取部署模型**] 清單已顯示 [**傳統**]，然後選取 [**建立**]。
   
    ![顯示 Azure 入口網站的螢幕擷取畫面，其中顯示已醒目提示新的 > 計算 > Windows Server 2012 R2 Datacenter 磚。](./media/virtual-networks-static-ip-classic-pportal/figure01.png)
3. 在 [建立 VM]  下，輸入要建立的 VM 名稱 (案例中為 *DNS01* )、本機系統管理員帳戶和密碼。
   
    ![顯示如何輸入 VM 的名稱、本機系統管理員使用者名稱和密碼來建立 VM 的螢幕擷取畫面。](./media/virtual-networks-static-ip-classic-pportal/figure02.png)
4. 選取 [**選用** 設定  >  **網路**  >  **虛擬網路**]，然後選取 [ **TestVNet**]。 如果不能使用 **TestVNet**，請確定您使用「美國中部」位置，並已建立本文開頭所描述的測試環境。
   
    ![顯示反白顯示 > 網路 > 虛擬網路 > TestVNet 選項的選項螢幕擷取畫面。](./media/virtual-networks-static-ip-classic-pportal/figure03.png)
5. 在 [網路] 下，請確定目前選取的子網路是 FrontEnd，然後選取 [IP 位址]，在 [IP 位址指派] 底下，選取 [靜態]，然後 [IP 位址] 輸入 192.168.1.101，如下所示。
   
    ![顯示您輸入靜態 IP 位址的 [IP 位址] 欄位的螢幕擷取畫面。](./media/virtual-networks-static-ip-classic-pportal/figure04.png)    
6. 在 [IP 位址] 下選取 [確定]、在 [網路] 下選取 [確定]，然後在 [選用設定] 下選取 [確定]。
7. 在 [建立 VM] 下，選取 [建立]。 請注意您儀表板中下方顯示的磚：
   
    ![顯示 [建立 Windows Server 2012 R2 Datacenter] 磚的螢幕擷取畫面。](./media/virtual-networks-static-ip-classic-pportal/figure05.png)

## <a name="how-to-retrieve-static-private-ip-address-information-for-a-vm"></a>如何擷取 VM 的靜態私人 IP 位址資訊
若要檢視使用上述步驟建立之 VM 的靜態私人 IP 位址資訊，請執行下列步驟。

1. 從 Azure 入口網站中，選取 **[流覽所有**  >  **虛擬機器] (傳統)**  >  **DNS01**  >  **所有設定**  >  **ip** 位址，並注意 ip 位址指派和 ip 位址，如下所示。
   
    ![在 Azure 入口網站中建立 VM](./media/virtual-networks-static-ip-classic-pportal/figure06.png)

## <a name="how-to-remove-a-static-private-ip-address-from-a-vm"></a>如何移除 VM 的靜態私人 IP 位址

在 [IP 位址] 下的 [IP 位址指派] 右側，選取 [動態]，選取 [儲存]，然後選取 [是]，如下圖所示：
   
![螢幕擷取畫面，顯示如何選取 IP 位址指派標籤右邊的動態，以移除 VM 的靜態私人 IP 位址。](./media/virtual-networks-static-ip-classic-pportal/figure07.png)

## <a name="how-to-add-a-static-private-ip-address-to-an-existing-vm"></a>如何將靜態私人 IP 位址新增至現有的 VM

1. 在前面顯示的 [IP 位址] 下，選取 [IP 位址指派] 右側的 [靜態]。
2. 在 [IP 位址] 輸入 192.168.1.101，選取 [儲存]，然後選取 [是]。

## <a name="set-ip-addresses-within-the-operating-system"></a>設定作業系統內的 IP 位址

除非必要，建議您不要靜態指派虛擬機器作業系統內已指派給 Azure 虛擬機器的私人 IP。 如果您確實手動設定作業系統內的私人 IP 位址，請確保該位址與指派給 Azure VM 的私人 IP 位址相同，否則可能會失去與虛擬機器的連線。 請勿手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。

## <a name="next-steps"></a>後續步驟
* 深入了解 [保留的公用 IP](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip) 位址。
* 深入了解 [執行個體層級公用 IP (ILPIP)](/previous-versions/azure/virtual-network/virtual-networks-instance-level-public-ip) 位址。
* 請參閱 [保留 IP REST API](/previous-versions/azure/reference/dn722420(v=azure.100))。