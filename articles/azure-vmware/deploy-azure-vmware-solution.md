---
title: 部署及設定 Azure VMware 解決方案
description: 了解如何使用在規劃階段中收集到的資訊來部署 Azure VMware 解決方案私人雲端。
ms.topic: tutorial
ms.date: 12/24/2020
ms.openlocfilehash: f2b6f3c4ad82117fee96e0c2e5973a7011384d48
ms.sourcegitcommit: 3c3ec8cd21f2b0671bcd2230fc22e4b4adb11ce7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2021
ms.locfileid: "98760880"
---
# <a name="deploy-and-configure-azure-vmware-solution"></a>部署及設定 Azure VMware 解決方案

在本文中，您將使用[規劃一節](production-ready-deployment-steps.md)中的資訊來部署 Azure VMware 解決方案。 

>[!IMPORTANT]
>如果您尚未定義資訊，請返回 [ [規劃] 區段](production-ready-deployment-steps.md) ，再繼續進行。

## <a name="register-the-resource-provider"></a>註冊資源提供者

[!INCLUDE [register-resource-provider-steps](includes/register-resource-provider-steps.md)]


## <a name="deploy-azure-vmware-solution"></a>部署 Azure VMware 解決方案

使用您在[規劃 Azure VMware 解決方案部署](production-ready-deployment-steps.md)文章中收集到的資訊：

>[!NOTE]
>若要部署 Azure VMware 解決方案，您必須是訂用帳戶中的最小參與者層級。

[!INCLUDE [create-avs-private-cloud-azure-portal](includes/create-private-cloud-azure-portal-steps.md)]

>[!NOTE]
>如需此步驟的端對端概觀，請檢視 [Azure VMware 解決方案：部署](https://www.youtube.com/embed/gng7JjxgayI)影片。

## <a name="create-the-jump-box"></a>建立跳躍箱

>[!IMPORTANT]
>如果您在 [建立私人雲端] 畫面上的初始佈建步驟期間，將 [虛擬網路] 選項保留空白，請先完成 [設定 VMware 私人雲端的網路功能](tutorial-configure-networking.md)教學課程 **之後**，再繼續進行本節。  

部署 Azure VMware 解決方案之後，您將建立虛擬網路的跳躍箱來連線至 vCenter 和 NSX。 設定 ExpressRoute 電路和 ExpressRoute Global Reach 之後，就不需要跳躍箱。  但是，在您的 Azure VMware 解決方案中觸達 vCenter 和 NSX 十分方便。  

:::image type="content" source="media/pre-deployment/jump-box-diagram.png" alt-text="建立 Azure VMware 解決方案跳躍箱" border="false" lightbox="media/pre-deployment/jump-box-diagram.png":::

若要在虛擬網路中建立虛擬機器 (VM) 在您于部署程式中 [識別或建立](production-ready-deployment-steps.md#attach-virtual-network-to-azure-vmware-solution)的虛擬網路中，請遵循下列指示： 

[!INCLUDE [create-avs-jump-box-steps](includes/create-jump-box-steps.md)]

## <a name="connect-to-a-virtual-network-with-expressroute"></a>透過 ExpressRoute 的虛擬網路

>[!IMPORTANT]
>如果您已在 Azure 的部署畫面中定義虛擬網路，請跳至下一節。

如果您未在部署步驟中定義虛擬網路，而您的目的是要將 Azure VMware 解決方案的 ExpressRoute 連線到現有的 ExpressRoute 閘道，請遵循下列步驟。

[!INCLUDE [connect-expressroute-to-vnet](includes/connect-expressroute-vnet.md)]

## <a name="verify-network-routes-advertised"></a>確認已公告網路路由

跳躍方塊位於虛擬網路中，Azure VMware 解決方案會透過其 ExpressRoute 線路進行連線。  在 Azure 中，移至跳躍箱的網路介面，並[檢視有效路由](../virtual-network/manage-route-table.md#view-effective-routes)。

在 [有效路由] 清單中，您應該會看到在 Azure VMware 解決方案部署過程中建立的網路。 您會在本文稍早的[部署步驟](#deploy-azure-vmware-solution)期間，看到多個衍生自[`/22`您所定義網路](production-ready-deployment-steps.md#ip-address-segment)的網路。

:::image type="content" source="media/pre-deployment/azure-vmware-solution-effective-routes.png" alt-text="確認從 Azure VMware 解決方案公告到 Azure 虛擬網路的網路路由" lightbox="media/pre-deployment/azure-vmware-solution-effective-routes.png":::

在此範例中，部署期間輸入的 10.74.72.0/22 網路會衍生 /24 網路。  如果您看到類似的內容，您可以連線到 Azure VMware 解決方案中的 vCenter。

## <a name="connect-and-sign-in-to-vcenter-and-nsx-t"></a>連線並登入 vCenter 和 NSX-T

登入您在先前步驟中建立的跳躍箱。 登入之後，請開啟網頁瀏覽器，並瀏覽至及登入 vCenter 和 NSX-T 管理主控台。  

您可以在 Azure 入口網站中識別 vCenter，以及 NSX-T 管理主控台的 IP 位址和認證。  選取您的私人雲端，然後在 **概觀** 檢視中選取 [識別 > 預設]。 

## <a name="create-a-network-segment-on-azure-vmware-solution"></a>在 Azure VMware 解決方案中建立網路區段

您可以使用 NSX-T 在您的 Azure VMware 解決方案環境中建立新的網路區段。  您已在 [ [規劃] 區段](production-ready-deployment-steps.md)中定義您想要建立的網路。  如果您尚未定義，請回到[規劃一節](production-ready-deployment-steps.md)，再繼續進行。

>[!IMPORTANT]
>請確定您所定義的 CIDR 網路位址區塊不會與 Azure 或內部部署環境中的任何項目重疊。  

遵循[在 Azure VMware 解決方案中建立 NSX-T 網路區段](tutorial-nsx-t-network-segment.md)教學課程，在 Azure VMware 解決方案中建立 NSX-T 網路區段。

## <a name="verify-advertised-nsx-t-segment"></a>驗證已公告的 NSX-T 區段

回到[確認公告的網路路由](#verify-network-routes-advertised)步驟。 您將會在清單中看到其他路由，代表您在上一個步驟中建立的網路區段。  

針對虛擬機器，您將指派您在「在 [Azure VMware 解決方案上建立網路區段](#create-a-network-segment-on-azure-vmware-solution) 」步驟中建立的區段。  

因為需要 DNS，請識別您想要使用的 DNS 伺服器。  

- 如果您已設定 ExpressRoute Global Reach，請使用您要在內部部署使用的任何 DNS 伺服器。  
- 如果您在 Azure 中具有 DNS 伺服器，請加以使用。  
- 如果您還沒有，請使用跳躍箱在使用的任何 DNS 伺服器。

>[!NOTE]
>此步驟是用來識別 DNS 伺服器，且在此步驟中不會進行任何設定。

## <a name="optional-provide-dhcp-services-to-nsx-t-network-segment"></a>(選擇性) 提供 DHCP 服務至 NSX-T 網路區段

如果您打算在您的 NSX-T 區段上使用 DHCP，請繼續進行本節。 否則，請跳至[在 NSX-T 網路區段上新增 VM](#add-a-vm-on-the-nsx-t-network-segment)一節。  

現在您已建立您的 NSX-T 網路區段，您可以透過兩種方式來建立和管理 Azure VMware 解決方案中的 DHCP：

* 如果您是使用 NSX-T 來裝載 DHCP 伺服器，您必須[建立 DHCP 伺服器](manage-dhcp.md#create-a-dhcp-server)並[轉送到該伺服器](manage-dhcp.md#create-dhcp-relay-service)。 
* 如果您在網路中使用第三方外部 DHCP 伺服器，您必須[建立 DHCP 轉送服務](manage-dhcp.md#create-dhcp-relay-service)。  針對此選項，[僅進行轉送組態](manage-dhcp.md#create-dhcp-relay-service)。


## <a name="add-a-vm-on-the-nsx-t-network-segment"></a>在 NSX-T 網路區段上新增 VM

在您的 Azure VMware 解決方案 vCenter 中，部署 VM 並使用它來確認從您的 Azure VMware 解決方案網路連線到：

- 網際網路
- Azure 虛擬網路
- 內部部署。  

如同在任何 vSphere 環境中一樣部署 VM。  將 VM 連接到您先前在 NSX-T 中建立的其中一個網路區段。  

>[!NOTE]
>如果您設定了 DHCP 伺服器，就會從其取得 VM 的網路組態 (請記得設定範圍)。  如果您要以靜態方式設定，請照常設定。

## <a name="test-the-nsx-t-segment-connectivity"></a>測試 NSX-T 區段連線能力

登入在上一個步驟中建立的 VM，並確認連線能力；

1. Ping 網際網路上的 IP。
2. 在網頁瀏覽器中，移至網際網路網站。
3. 偵測位於 Azure 虛擬網路的跳躍箱。

Azure VMware 解決方案現在已啟動並在執行中，而且您已成功建立與 Azure 虛擬網路和網際網路的連線能力。

## <a name="next-steps"></a>後續步驟

在下一節中，您會透過 ExpressRoute 將 Azure VMware 解決方案連線至您的內部部署網路。
> [!div class="nextstepaction"]
> [將 Azure VMware 解決方案連線到您的內部部署環境](azure-vmware-solution-on-premises.md)
