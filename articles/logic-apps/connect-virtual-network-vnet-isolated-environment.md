---
title: 從 Azure Logic Apps 連線到 Azure 虛擬網路
description: 若要從 Azure Logic Apps 存取 Azure 虛擬網路，您可以建立專用且隔離的私人整合服務環境，讓邏輯應用程式和其他資源與公用或「全域」 Azure 分開
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 09/25/2018
ms.openlocfilehash: 354c31014448b914b33d2bef5483efc78092f726
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47391916"
---
# <a name="create-isolated-environments-to-access-azure-virtual-networks-from-azure-logic-apps"></a>建立隔離的環境，以從 Azure Logic Apps 存取 Azure 虛擬網路

> [!NOTE]
> 這項功能目前處於「個人預覽版」階段。 若要要求存取，請在[這裡建立您的加入要求](https://aka.ms/iseprivatepreview)。

對於您邏輯應用程式與整合帳戶需要存取 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md)的整合案例，您可以建立[整合服務環境 (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)，以連結到您的虛擬網路，並將 Logic Apps 服務部署到您的網路。 當您建立邏輯應用程式與整合帳戶時，您可以選取此 ISE 作為其位置。 然後，您的邏輯應用程式與整合帳戶就可以直接存取虛擬網路中的資源，例如虛擬機器 (VM)、伺服器、系統及服務。 

![選取整合服務環境](./media/connect-virtual-network-vnet-isolated-environment/select-logic-app-integration-service-environment.png)

您的 ISE 是一種隔離的私人環境，它會使用與公用或「全域」Logic Apps 服務分開存在的專用儲存體和其他資源。 這種區隔也有助於減少其他 Azure 租用戶對您應用程式效能可能造成的任何影響。 

本文會示範如何執行這些工作：

* 設定 Azure 虛擬網路的權限，讓私人 Logic Apps 執行個體可以存取您的虛擬網路。

* 建立您的整合服務環境 (ISE)。 

* 建立可以在您 ISE 中執行的邏輯應用程式。 

* 在您的 ISE 中建立邏輯應用程式的整合帳戶。

如需整合服務環境的詳細資訊，請參閱[從隔離的 Azure Logic Apps 存取 Azure 虛擬網路資源](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 如果您沒有 Azure 虛擬網路，請了解如何[建立 Azure 虛擬網路](../virtual-network/quick-create-portal.md)。 

  > [!IMPORTANT]
  > 雖然您不需要 Azure 虛擬網路來建立您的環境，但您可以在建立該環境時，「只」選取虛擬網路作為環境的對等項目。 

* 若要讓邏輯應用程式直接存取您的 Azure 虛擬網路，請[設定角色型存取控制 (RBAC) 權限](#vnet-access)，讓 Logic Apps 服務具有存取虛擬網路的權限。 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

<a name="vnet-access"></a>

## <a name="set-virtual-network-permissions"></a>設定虛擬網路權限

當您建立整合服務環境時，可以選取 Azure 虛擬網路作為您環境的「對等項目」。 但是，您在建立您的 ISE 時，只能執行此步驟，或進行「對等互連」。 此關聯性可讓 Logic Apps 服務直接連線到該虛擬網路中的資源，並可讓您的環境存取這些資源。 

您必須在虛擬網路中設定角色型存取控制 (RBAC) 權限，才能選取您的虛擬網路。 若要完成這項工作，您必須將特定角色指派給 Azure Logic Apps 服務。

1. 在 [Azure 入口網站](https://portal.azure.com)中，尋找並選取您的虛擬網路。 

1. 在您虛擬網路的功能表中，選取 [存取控制 (IAM)]。 

1. 在 [存取控制] 下方，選取 [角色指派] (若尚未選取)。 在 [角色指派] 工具列上選擇 [新增]。 

   ![新增角色指派](./media/connect-virtual-network-vnet-isolated-environment/set-up-role-based-access-control-vnet.png)

1. 在 [新增權限] 窗格中，為 Azure Logic Apps 服務設定此資料表中的每個角色。 請確定您在完成每個角色之後選擇 [儲存]：

   | 角色 | 存取權指派對象 | 選取 | 
   |------|------------------|--------|
   | **網路參與者** | **Azure AD 使用者、群組或應用程式** | 輸入 **Azure Logic Apps**。 在成員清單出現之後，請選取相同值。 <p>**提示**：如果您找不到此服務，請輸入 Logic Apps 服務的應用程式識別碼：`7cd684f4-8a78-49b0-91ec-6a35d38739ba` | 
   | **傳統參與者** | **Azure AD 使用者、群組或應用程式** | 輸入 **Azure Logic Apps**。 在成員清單出現之後，請選取相同值。 <p>**提示**：如果您找不到此服務，請輸入 Logic Apps 服務的應用程式識別碼：`7cd684f4-8a78-49b0-91ec-6a35d38739ba` | 
   |||| 

   例如︰

   ![新增權限](./media/connect-virtual-network-vnet-isolated-environment/add-contributor-roles.png)

   如需對等互連所需角色權限的詳細資訊，請參閱[建立、變更或刪除虛擬網路對等互連中的權限一節](../virtual-network/virtual-network-manage-peering.md#permissions)。 

如果您的虛擬網路已透過 Azure ExpressRoute、Azure 點對站 VPN 或 Azure 站對站 VPN 連線，請繼續進行下一節，以便新增所需的閘道子網路。 否則，請繼續[建立您的環境](#create-environment)。

<a name="add-gateway-subnet"></a>

## <a name="add-gateway-subnet-for-virtual-networks-with-expressroute-or-vpns"></a>使用 ExpressRoute 或 VPN 新增虛擬網路的閘道子網路

完成上述步驟之後，若要讓您的整合服務環境 (ISE) 存取透過 [Azure ExpressRoute](../expressroute/expressroute-introduction.md)、[Azure 點對站 VPN](../vpn-gateway/point-to-site-about.md) 或 [Azure 站對站 VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) 連線的 Azure 虛擬網路，您也必須在虛擬網路中新增[*閘道子網路*](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsub)：

1. 在 [Azure 入口網站](https://portal.azure.com)中，尋找並選取您的虛擬網路。 在您虛擬網路的功能表中，選取 [子網路]，然後選擇 [閘道子網路] > [確定]。

   ![新增閘道子網路](./media/connect-virtual-network-vnet-isolated-environment/add-gateway-subnet.png)

1. 現在，建立[路由表](../virtual-network/manage-route-table.md)，您會將其與稍早建立的閘道子網路建立關聯。

   1. 在 Azure 主功能表上，選取 **[建立資源]** > 
    **[網路]** > **[路由表]**。

      ![建立路由表](./media/connect-virtual-network-vnet-isolated-environment/create-route-table.png)

   1. 提供路由表的相關資訊，例如名稱、要使用的 Azure 訂用帳戶、Azure 資源群組和位置。 請確定 [BGP 路由傳播] 屬性設定為 [啟用]，然後選擇 [建立]。

      ![提供路由表詳細資料](./media/connect-virtual-network-vnet-isolated-environment/enter-route-table-information.png)

   1. 在路由表的功能表中，選取 [子網路]，然後選擇 [關聯]。 

      ![將路由表連接到子網路](./media/connect-virtual-network-vnet-isolated-environment/associate-route-table.png)

   1. 選取 [虛擬網路]，然後選取您的虛擬網路。
   
   1. 選取 [子網路]，然後選取您先前建立的閘道子網路。

   1. 完成時，選擇 [確定]。

1. 如果您有點對站 VPN，也請完成下列步驟：

   1. 在 Azure 中，尋找並選取您的虛擬網路閘道資源。

   1. 在閘道的功能表上，選取 [點對站設定]。 
   然後選擇 [下載 VPN 用戶端]，讓您有最新的 VPN 用戶端設定。

      ![下載最新的 VPN 用戶端](./media/connect-virtual-network-vnet-isolated-environment/download-vpn-client.png)

您現在已針對使用 ExpressRoute、點對站 VPN 或站對站 VPN 的虛擬網路完成閘道子網路的設定。 若要繼續建立您的整合服務環境，請遵循下一步。

<a name="create-environment"></a>

## <a name="create-your-ise"></a>建立您的 ISE

若要建立您的整合服務環境 (ISE)，請遵循下列步驟：

1. 在 [Azure 入口網站](https://portal.azure.com)中，於 Azure 主功能表上選取 [建立資源]。

   ![建立新資源](./media/connect-virtual-network-vnet-isolated-environment/find-integration-service-environment.png)

1. 在搜尋方塊中，輸入「整合服務環境」作為篩選條件。
從 [結果] 清單中，選取 [整合服務環境 (預覽)]，然後選擇 [建立]。

   ![選取 [整合服務環境]](./media/connect-virtual-network-vnet-isolated-environment/select-integration-service-environment.png)

   ![選擇 [建立]](./media/connect-virtual-network-vnet-isolated-environment/create-integration-service-environment.png)

1. 針對您的環境提供以下詳細資料：

   ![提供環境詳細資料](./media/connect-virtual-network-vnet-isolated-environment/integration-service-environment-details.png)

   | 屬性 | 必要 | 值 | 說明 |
   |----------|----------|-------|-------------|
   | **名稱** | 是 | <*environment-name*> | 提供給環境的名稱 | 
   | **訂用帳戶** | 是 | <*Azure-subscription-name*> | 要用於環境的 Azure 訂用帳戶 | 
   | **資源群組** | 是 | <*Azure-resource-group-name*> | 您要用來建立環境的 Azure 資源群組 |
   | **位置** | 是 | <*Azure-datacenter-region*> | 要用來存放環境相關資訊的 Azure 資料中心區域 |
   | **對等 VNET** | 否 | <*Azure-VNET-name*> | Azure 虛擬網路，其要以「對等項目」形式與您的環境建立關聯，讓該環境中的邏輯應用程式可以存取虛擬網路。 在您能夠建立此關聯性之前，請確定您已經[在虛擬網路中為 Azure Logic Apps 設定角色型存取控制](#vnet-access)。 <p>**重要**：雖然不需要虛擬網路，但您可以在建立環境時「只」選取虛擬網路。 | 
   | **對等互連名稱** | 「是」且具有選取的虛擬網路 | <*peering-name*> | 提供給對等關聯性的名稱 | 
   | **VNET IP 範圍** | 「是」且具有選取的虛擬網路 | <*IP-address-range*> | 要用於建立環境資源的 IP 位址範圍。 此範圍必須使用[無類別網域間路由選擇 (CIDR) 格式](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) (例如 10.0.0.1/16)，且需要類別 B 位址空間。 此範圍不得存在於 [對等 VNET] 屬性中所選取虛擬網路的位址空間，也不得存在於透過對等互連或閘道連接對等網路的任何其他私人 IP 位址。 <p><p>**重要**：您在建立環境之後「無法變更」此位址範圍。 |
   |||||
   
1. 完成之後，請選擇 [建立]。 

   Azure 會開始部署您的環境，但此程序可能需要「長達兩小時」的時間才能完成。 
   若要檢查部署狀態，請在 Azure 工具列上選擇通知圖示，這會開啟 [通知] 窗格。

   ![檢查部署狀態](./media/connect-virtual-network-vnet-isolated-environment/environment-deployment-status.png)

   成功完成部署時，Azure 就會顯示此通知：

   ![部署成功](./media/connect-virtual-network-vnet-isolated-environment/deployment-success.png)

1. 如果 Azure 在部署完成之後不會自動移至您的環境，且若要檢視您的環境，請選擇 [移至資源]。  

<a name="create-logic-apps-environment"></a>

## <a name="create-logic-app---ise"></a>建立邏輯應用程式 - ISE

若要建立使用整合服務環境 (ISE) 的邏輯應用程式，請遵循[如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)中的一般步驟，但具有以下差異和考量： 

* 當您建立邏輯應用程式時，[位置] 屬性會在 [整合服務環境] 下方列出您的 ISE 以及可用的區域。 請選取您 ISE，而不是區域，例如：

  ![選取整合服務環境](./media/connect-virtual-network-vnet-isolated-environment/create-logic-app-with-integration-service-environment.png)

* 您可以使用同一個內建項目 (例如，HTTP 觸發程序或動作)，以便在與父邏輯應用程式相同的 ISE 中執行。 具有 **ISE** 標籤的連接器也會在與父邏輯應用程式相同的 ISE 中執行。 不具備 **ISE** 標籤的連接器則會在全域 Logic Apps 服務中執行。

  ![選取 ISE 連接器](./media/connect-virtual-network-vnet-isolated-environment/select-ise-connectors.png)

* 如果您先前設定的 ISE 使用 Azure 虛擬網路作為對等項目，則 ISE 中的邏輯應用程式可以直接存取該虛擬網路中的資源。 針對 ISE 所連結虛擬網路中的內部部署系統，邏輯應用程式可透過使用以下任何項目，直接存取這些系統： 

  * 該系統的 ISE 連接器，例如 SQL Server

  * HTTP 動作 

  * 自訂連接器

  針對未在虛擬網路內或不具備 ISE 連接器的內部部署系統，您仍然可以在[設定及使用內部部署的資料閘道](../logic-apps/logic-apps-gateway-install.md)之後進行連線。

<a name="create-integration-account-environment"></a>

## <a name="create-integration-account---ise"></a>建立整合帳戶 - ISE

若要在整合服務環境 (ISE) 中搭配使用整合帳戶與邏輯應用程式，該整合帳戶必須使用與邏輯應用程式「相同的環境」。 ISE 中的邏輯應用程式只能參考相同 ISE 中的整合帳戶。 

若要建立使用 ISE 的整合帳戶，請遵循[如何建立整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)中的一般步驟，但 [位置] 屬性除外，它現在會在 [整合服務環境] 下方列出您的 ISE 以及可用區域。 請選取您的 ISE，而不是區域，例如：

![選取整合服務環境](./media/connect-virtual-network-vnet-isolated-environment/create-integration-account-with-integration-service-environment.png)

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 <a href="https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps" target="_blank">Azure Logic Apps 論壇</a>。
* 若要提交或票選功能構想，請造訪 <a href="http://aka.ms/logicapps-wish" target="_blank">Logic Apps 使用者意見反應網站</a>。

## <a name="next-steps"></a>後續步驟

* 深入了解 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md)
* 了解 [Azure 服務的虛擬網路整合](../virtual-network/virtual-network-for-azure-services.md)
