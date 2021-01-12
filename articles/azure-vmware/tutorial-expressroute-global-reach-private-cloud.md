---
title: 教學課程 - 將內部部署環境對等互連至私人雲端
description: 了解如何在 Azure VMware 解決方案中建立與私人雲端對等互連的 ExpressRoute Global Reach。
ms.topic: tutorial
ms.date: 1/5/2021
ms.openlocfilehash: 613aece6ed548f70840349e017de4416883d6cf3
ms.sourcegitcommit: 67b44a02af0c8d615b35ec5e57a29d21419d7668
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/06/2021
ms.locfileid: "97913153"
---
# <a name="tutorial-peer-on-premises-environments-to-a-private-cloud"></a>教學課程：將內部部署環境對等互連至私人雲端

ExpressRoute Global Reach 可將您的內部部署環境連線至 Azure VMware 解決方案私人雲端。 在私人雲端 ExpressRoute 線路與內部部署環境的現有 ExpressRoute 連線之間，會建立 ExpressRoute Global Reach 連線。 

您在[設定 Azure 對私人雲端網路](tutorial-configure-networking.md)時所使用的 ExpressRoute 線路會要求您建立並使用授權金鑰。  您已使用 ExpressRoute 線路中的一個授權金鑰，而在第二個教學課程中，您將建立第二個授權金鑰，以便與內部部署 ExpressRoute 線路對等互連。

在本教學課程中，您會了解如何：

> [!div class="checklist"]
> * 為 _線路 2_ (私人雲端 ExpressRoute 線路) 建立第二個授權金鑰
> * 在 _線路 1_ 的訂用帳戶中，使用 [Azure 入口網站](#azure-portal-method)或 [Cloud Shell 中的 Azure CLI](#azure-cli-in-a-cloud-shell-method) 方法，啟用內部部署對私人雲端的 ExpressRoute Global Reach 對等互連


## <a name="before-you-begin"></a>開始之前

在使用 ExpressRoute Global Reach 啟用兩個 ExpressRoute 線路之間的連線之前，請檢閱有關如何[在不同 Azure 訂用帳戶中啟用連線](../expressroute/expressroute-howto-set-global-reach-cli.md#enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions)的文件。  


## <a name="prerequisites"></a>必要條件

- 已建立進出於 Azure VMware 解決方案私人雲端的連線，且其 ExpressRoute 線路已與 Azure 虛擬網路 (VNet) 中的 ExpressRoute 閘道對等互連 – 這是對等互連程序中的 _線路 2_。  
- 不同的運作中 ExpressRoute 線路，用以將內部部署環境連線至 Azure – 從對等互連程序的角度來看，這是 _線路 1_。
- /29 的非重疊[網路位址區塊](../expressroute/expressroute-routing.md#ip-addresses-used-for-peerings)，用於進行 ExpressRoute Global Reach 對等互連。
- 請確定包含 ExpressRoute 提供者服務的所有路由器都支援 4 個位元組的自發系統編號 (ASN)。 Azure VMware 解決方案會使用 4 位元組的公用 ASN 公告路由。

> [!TIP]
> 在這些必要條件的內容中，您的內部部署 ExpressRoute 線路是 _線路 1_，而您的私人雲端 ExpressRoute 線路會在不同的訂用帳戶中，並標示為 _線路 2_。 


## <a name="create-an-expressroute-authorization-key-in-the-private-cloud"></a>在私人雲端中建立 ExpressRoute 授權金鑰

1. 從私人雲端 [概觀]的 [管理] 底下，選取 [連線] > [ExpressRoute] > [要求授權金鑰]。

   :::image type="content" source="media/expressroute-global-reach/start-request-auth-key.png" alt-text="選取 [連線] > [ExpressRoute] > [要求授權金鑰] 以開始新的要求。":::

2. 輸入授權金鑰的名稱，然後選取 [建立]。 

   :::image type="content" source="media/expressroute-global-reach/create-global-reach-auth-key.png" alt-text="選取 [建立] 以建立新的授權金鑰。":::

   建立之後，新的金鑰會出現在私人雲端的授權金鑰清單中。 

   :::image type="content" source="media/expressroute-global-reach/show-global-reach-auth-key.png" alt-text="確認新的授權金鑰出現在私人雲端的金鑰清單中。":::

3. 記下授權金鑰和 ExpressRoute 識別碼，以及 /29 位址區塊。 您將在下一個步驟中使用這些資料來完成對等互連。 

## <a name="peer-private-cloud-to-on-premises-using-authorization-key"></a>使用授權金鑰將私人雲端對等互連至內部部署環境

您已建立私人雲端 ExpressRoute 線路的授權金鑰，接下來可以讓其與您的內部部署 ExpressRoute 線路對等互連。  對等互連可使用 [Azure 入口網站](#azure-portal-method)或 [Cloud Shell 中的 Azure CLI](#azure-cli-in-a-cloud-shell-method)，從內部部署 ExpressRoute 線路方面來完成。 使用這兩種方法時，您都會使用私人雲端 ExpressRoute 線路的資源識別碼和授權金鑰來完成對等互連。

### <a name="azure-portal-method"></a>Azure 入口網站方法

1. 使用與內部部署 ExpressRoute 線路相同的訂用帳戶登入 [Azure 入口網站](https://portal.azure.com)。

1. 從私人雲端 [概觀] 的 [管理] 底下，選取 [連線] > [ExpressRoute Global Reach] > [新增]。

   :::image type="content" source="./media/expressroute-global-reach/expressroute-global-reach-tab.png" alt-text="從功能表中依序選取 [連線]、[ExpressRoute Global Reac] 索引標籤和 [新增]。":::

1. 您可以執行下列其中一個選項，以建立內部部署雲端連線：

   - 從清單中選取 ExpressRoute 線路。
   - 如果您有線路識別碼，請將其複製並貼上。

1. 選取 [連接]  。 新的連線會顯示在內部部署雲端連線清單中。  

>[!TIP]
>您可以選取 [其他]，以刪除或中斷清單中的連線。  
>
> :::image type="content" source="./media/expressroute-global-reach/on-premises-connection-disconnect.png" alt-text="中斷或刪除內部部署連線":::

### <a name="azure-cli-in-a-cloud-shell-method"></a>Cloud Shell 中的 Azure CLI 方法

我們已透過特定詳細資料和範例補強 [CLI 命令](../expressroute/expressroute-howto-set-global-reach-cli.md)，以協助您設定內部部署環境與 Azure VMware 解決方案私人雲端之間的 ExpressRoute Global Reach 對等互連。  

> [!TIP]  
> 為了簡單起見，在 Azure CLI 的命令輸出中，這些指示可能會使用 [`–query` 引數來執行 JMESPath 查詢，從而只顯示所需的結果](/cli/azure/query-azure-cli)。


1. 使用與內部部署 ExpressRoute 線路相同的訂用帳戶登入 Azure 入口網站，然後開啟 Cloud Shell。 讓殼層保持是 Bash。
 
   :::image type="content" source="media/expressroute-global-reach/open-cloud-shell.png" alt-text="登入 Azure 入口網站並開啟 Cloud Shell。":::
 
2. 輸入 Azure CLI 命令以建立對等互連。 使用您的特定資訊和資源識別碼、授權金鑰和 /29 CIDR 網路區塊。 

   此影像顯示您將使用的命令範例，以及指出對等互連成功的輸出。 範例命令是以[「在不同 Azure 訂用帳戶中啟用 ExpressRoute 線路之間的連線」的步驟 3](../expressroute/expressroute-howto-set-global-reach-cli.md#enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions) 中所使用的命令作為基礎。

   :::image type="content" source="media/expressroute-global-reach/azure-command-with-results.png" alt-text="在 Cloud Shell 中使用 Azure CLI 命令建立 ExpressRoute Global Reach 對等互連。":::
 
   您可以透過 ExpressRoute Global Reach 對等互連，從內部部署環境連線至您的私人雲端。

> [!TIP]
> 您可以遵循[停用內部部署網路之間的連線](../expressroute/expressroute-howto-set-global-reach-cli.md#disable-connectivity-between-your-on-premises-networks)來刪除您剛才建立的對等互連。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何建立私人雲端 ExpressRoute 線路的第二個授權金鑰。 您也已經了解如何啟用內部部署對私人雲端 ExpressRoute Global Reach 的對等互連。 

繼續進行下一個教學課程，了解如何為您的 Azure VMware 解決方案私人雲端部署及設定 VMware HCX 解決方案。

> [!div class="nextstepaction"]
> [部署及設定 VMWare HCX](tutorial-deploy-vmware-hcx.md)


<!-- LINKS - external-->

<!-- LINKS - internal -->