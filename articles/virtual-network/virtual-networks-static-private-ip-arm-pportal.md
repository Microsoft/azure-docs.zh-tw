---
title: 設定 VM 的私人 IP 位址 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用 Azure 入口網站設定虛擬機器的私人 IP 位址。
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: 11245645-357d-4358-9a14-dd78e367b494
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2016
ms.author: jdial
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ff9d025980f80cb77246ea12dbf2e9bcedd73f86
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38705879"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal"></a>使用 Azure 入口網站設定虛擬機器的私人 IP 位址

> [!div class="op_single_selector"]
> * [Azure 入口網站](virtual-networks-static-private-ip-arm-pportal.md)
> * [PowerShell](virtual-networks-static-private-ip-arm-ps.md)
> * [Azure CLI](virtual-networks-static-private-ip-arm-cli.md)
> * [Azure 入口網站 (傳統)](virtual-networks-static-private-ip-classic-pportal.md)
> * [PowerShell (傳統)](virtual-networks-static-private-ip-classic-ps.md)
> * [Azure CLI (傳統)](virtual-networks-static-private-ip-classic-cli.md)


[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文涵蓋之內容包括資源管理員部署模型。 您也可以 [管理傳統部署模型中的靜態私人 IP 位址](virtual-networks-static-private-ip-classic-pportal.md)。

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

下列範例步驟預期已經建立簡單的環境。 如果您想要執行如本文件中所示的步驟，請先建置[建立虛擬網路](quick-create-portal.md)中所述的測試環境。

## <a name="how-to-create-a-vm-for-testing-static-private-ip-addresses"></a>如何建立用來測試靜態私人 IP 位址的 VM
您無法在資源管理員部署模式中建立 VM 期間，使用 Azure 入口網站設定靜態私人 IP 位址。 您必須先建立 VM，才能將其私人 IP 設定為靜態。

若要在名為 TestVNet 之 VNet 的 FrontEnd 子網路中建立名為 DNS01 的 VM，請遵循下列步驟：

1. 透過瀏覽器瀏覽至 http://portal.azure.com，並視需要使用您的 Azure 帳戶登入。
2. 按一下 [建立資源] > [計算] > [Windows Server 2012 R2 Datacenter]，注意 [選取部署模型] 清單已經顯示 [Resource Manager]，然後按一下 [建立]，如下圖所示。
   
    ![在 Azure 入口網站中建立 VM](./media/virtual-networks-static-ip-arm-pportal/figure01.png)
3. 在 [基本概念] 窗格中，輸入要建立之 VM 的名稱 (此案例中為 DNS01)、本機系統管理員帳戶和密碼，如下圖所示。
   
    ![[基本概念] 窗格](./media/virtual-networks-static-ip-arm-pportal/figure02.png)
4. 請確定選取的 [位置] 為「美國中部」，按一下 [資源群組] 底下的 [選取現有項目]，然後再按一次 [資源群組]，按一下 [TestRG]，再按一下 [確定]。
   
    ![[基本概念] 窗格](./media/virtual-networks-static-ip-arm-pportal/figure03.png)
5. 在 [選擇大小] 窗格中，選取 [A1 標準]，然後按一下 [選取]。
   
    ![[選擇大小] 窗格](./media/virtual-networks-static-ip-arm-pportal/figure04.png)    
6. 在 [設定] 窗格中，確定屬性都已設定為下列值，然後按一下 [確定]。
   
    -**儲存體帳戶**：vnetstorage
   
   * **網路**：TestVNet
   * **子網路**：FrontEnd
     
     ![[選擇大小] 窗格](./media/virtual-networks-static-ip-arm-pportal/figure05.png)     
7. 在 [摘要] 窗格中，按一下 [確定]。 請注意儀表板中顯示的下列圖格。
   
    ![在 Azure 入口網站中建立 VM](./media/virtual-networks-static-ip-arm-pportal/figure06.png)

建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-portal.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。 請勿手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。

## <a name="how-to-retrieve-static-private-ip-address-information-for-a-vm"></a>如何擷取 VM 的靜態私人 IP 位址資訊
若要檢視使用上述步驟建立之 VM 的靜態私人 IP 位址資訊，請執行下列步驟。

1. 從 Azure 入口網站，按一下 [瀏覽全部] > [虛擬機器] > [DNS01] > [所有設定] > [網路介面]，然後按一下所列出的唯一網路介面。
   
    ![部署 VM 磚](./media/virtual-networks-static-ip-arm-pportal/figure07.png)
2. 在 [網路介面] 窗格中，按一下 [所有設定] > [IP 位址]，注意 [指派] 與 [IP 位址] 的值。
   
    ![部署 VM 磚](./media/virtual-networks-static-ip-arm-pportal/figure08.png)

## <a name="how-to-add-a-static-private-ip-address-to-an-existing-vm"></a>如何將靜態私人 IP 位址新增至現有的 VM
若要將靜態私人 IP 位址新增至使用上述步驟建立之 VM，請遵循下列步驟：

1. 從上方顯示的 [IP 位址] 窗格中，按一下 [指派] 底下的[靜態]。
2. [IP 位址] 輸入 192.168.1.101，然後按一下 [儲存]。
   
    ![在 Azure 入口網站中建立 VM](./media/virtual-networks-static-ip-arm-pportal/figure09.png)

> [!NOTE]
> 如果按一下 [儲存] 之後，您注意到指派仍然設定為 [動態]，這表示您輸入的 IP 位址已在使用中。 請嘗試不同的 IP 位址。
> 
> 

建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-portal.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。 請勿手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。

## <a name="how-to-remove-a-static-private-ip-address-from-a-vm"></a>如何移除 VM 的靜態私人 IP 位址
若要移除上述所建立 VM 的靜態私人 IP 位址，請完成下列步驟：

從上方顯示的 [IP 位址] 窗格中，按一下 [指派] 底下的[動態]，然後按一下 [儲存]。

## <a name="set-ip-addresses-within-the-operating-system"></a>設定作業系統內的 IP 位址

建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-portal.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。 請勿手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。

## <a name="next-steps"></a>後續步驟

了解如何管理 [IP 位址設定](virtual-network-network-interface-addresses.md)。

