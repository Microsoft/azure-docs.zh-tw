---
title: 建立具有靜態公用 IP 位址的 VM - Azure 入口網站 | Microsoft Docs
description: 了解如何使用 Azure 入口網站建立具有靜態公用 IP 位址的 VM。
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: e9546bcc-f300-428f-b94a-056c5bd29035
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2016
ms.author: jdial
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 524293f9a1ded73ee7cb6ba4f53208a9f9c54ffa
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38670979"
---
# <a name="create-a-vm-with-a-static-public-ip-address-using-the-azure-portal"></a>使用 Azure 入口網站建立具有靜態公用 IP 位址的 VM

> [!div class="op_single_selector"]
> * [Azure 入口網站](virtual-network-deploy-static-pip-arm-portal.md)
> * [PowerShell](virtual-network-deploy-static-pip-arm-ps.md)
> * [Azure CLI](virtual-network-deploy-static-pip-arm-cli.md)
> * [PowerShell (傳統)](virtual-networks-reserved-public-ip.md)

[!INCLUDE [virtual-network-deploy-static-pip-intro-include.md](../../includes/virtual-network-deploy-static-pip-intro-include.md)]

> [!NOTE]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用 Resource Manager 部署模型，Microsoft 建議大部分的新部署使用此模型，而不是傳統部署模型。

[!INCLUDE [virtual-network-deploy-static-pip-scenario-include.md](../../includes/virtual-network-deploy-static-pip-scenario-include.md)]

## <a name="create-a-vm-with-a-static-public-ip"></a>使用靜態公用 IP 建立 VM

若要在 Azure 入口網站中建立具有靜態公用 IP 位址的 VM，請完成下列步驟：

1. 透過瀏覽器瀏覽至 [Azure 入口網站](https://portal.azure.com) ，並視需要使用您的 Azure 帳戶登入。
2. 在入口網站左上角，按一下 [建立資源]>>[計算]>[Windows Server 2012 R2 Datacenter]。
3. 在 [選取部署模型] 清單中，選取 [Resource Manager]，然後按一下 [建立]。
4. 在 [基本] 窗格中，輸入如下所示的 VM 資訊，然後按一下 [確定]。
   
    ![Azure 入口網站 - 基本](./media/virtual-network-deploy-static-pip-arm-portal/figure1.png)
5. 在 [選擇大小] 窗格中，按一下如下所示的 [A1 標準]，然後按一下 [選取]。
   
    ![Azure 入口網站 - 選擇大小](./media/virtual-network-deploy-static-pip-arm-portal/figure2.png)
6. 在 [設定] 窗格中，按一下 [公用 IP 位址]，接著在 [建立公用 IP 位址] 窗格的 [指派] 底下，按一下 [靜態]，如下所示。 然後按一下 [確定] 。
   
    ![Azure 入口網站 - 建立公用 IP 位址](./media/virtual-network-deploy-static-pip-arm-portal/figure3.png)
7. 在 [設定] 窗格中，按一下 [確定]。
8. 檢閱 [摘要] 窗格，然後按一下 [確定]。
   
    ![Azure 入口網站 - 建立公用 IP 位址](./media/virtual-network-deploy-static-pip-arm-portal/figure4.png)
9. 請注意您儀表板中新的磚。
   
    ![Azure 入口網站 - 建立公用 IP 位址](./media/virtual-network-deploy-static-pip-arm-portal/figure5.png)
10. 建立 VM 後，[設定]  窗格隨即出現，如下所示：
    
    ![Azure 入口網站 - 建立公用 IP 位址](./media/virtual-network-deploy-static-pip-arm-portal/figure6.png)

## <a name="set-ip-addresses-within-the-operating-system"></a>設定作業系統內的 IP 位址

您絕不應手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。 建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-portal.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。

## <a name="next-steps"></a>後續步驟

任何網路流量均可流入和流出本文所建立的 VM。 您可以在網路安全性群組內定義輸入和輸出安全性規則，以限制網路介面和 (或) 子網路可以流入和流出的流量。 若要深入了解網路安全性群組，請參閱[網路安全性群組概觀](security-overview.md)。