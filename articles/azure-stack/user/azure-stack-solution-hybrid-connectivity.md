---
title: 使用 Azure 和 Azure Stack 設定混合式雲端連線 | Microsoft Docs
description: 了解如何使用 Azure 和 Azure Stack 設定混合式雲端連線。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/28/2018
ms.author: mabrigg
ms.reviewer: anajod
ms.openlocfilehash: a219ec1797ff74bd1a9ff458e34650d1545f1172
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47585776"
---
# <a name="tutorial-configure-hybrid-cloud-connectivity-with-azure-and-azure-stack"></a>教學課程：使用 Azure 和 Azure Stack 設定混合式雲端連線

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以在全域 Azure 和 Azure Stack 中使用混合式連線模式安全地存取資源。

在本教學課程中，您將建置範例環境，用以：

> [!div class="checklist"]
> - 使內部部署資料符合隱私權或法規需求，但具有全域 Azure 資源的存取權。
> - 在全域 Azure 中使用雲端調整應用程式部署和資源的同時，維護舊版系統。

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack 是 Azure 的延伸模組。 Azure Stack 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式的設計考量](https://aka.ms/hybrid-cloud-applications-pillars)技術白皮書檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，讓生產環境遇到最少的挑戰。


## <a name="prerequisites"></a>必要條件

建置混合式連線部署時需要幾項元件。 其中有些元件可能需要一些時間來準備，所以您必須進行規劃。

**Azure Stack**

Azure OEM/硬體合作夥伴可部署生產 Azure Stack，而所有使用者均可部署 Azure Stack 開發套件 (ASDK)。

**Azure Stack 元件**

Azure Stack 操作員必須部署 App Service、建立方案和供應項目、建立租用戶訂用帳戶，以及新增 Windows Server 2016 映像。 如果您已擁有其中部分元件，請在開始本教學課程之前，先確定它們符合需求。

本教學課程假設您擁有 Azure 和 Azure Stack 的一些基本知識。 若要在開始本教學課程前深入了解，請閱讀下列文章：

 - [Azure 簡介](https://azure.microsoft.com/overview/what-is-azure/)
 - [Azure Stack 重要概念](https://docs.microsoft.com/azure/azure-stack/azure-stack-key-features)

### <a name="azure"></a>Azure

 - 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
 - 在 Azure 中建立 [Web 應用程式](https://docs.microsoft.com/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?view=vsts&tabs=vsts#create-an-azure-web-app-using-the-portal)。 請記下 Web 應用程式 URL，因為您在本教學課程中需要它。

### <a name="azure-stack"></a>Azure Stack

 - 使用生產 Azure Stack，或從 https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1 部署 Azure Stack 開發套件。
   >[!Note]
   >部署 ASDK 可能需要 7 小時，因此請加以規劃。

 - 將 [App Service](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-deploy) PaaS 服務部署至 Azure Stack。
 - 在 Azure Stack 環境中[建立計劃和供應項目](https://docs.microsoft.com/azure/azure-stack/azure-stack-plan-offer-quota-overview)。
 - 在 Azure Stack 環境中建立[租用戶訂用帳戶](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm)。

### <a name="before-you-begin"></a>開始之前

先確認您符合下列準則，再開始設定混合式雲端連線：

 - 您需要 VPN 裝置對外開放的公用 IPv4 位址。 此 IP 位址不能位於 NAT 後方。
 - 所有資源皆部署於相同的區域/位置。

#### <a name="tutorial-example-values"></a>教學課程的範例值

本教學課程中的範例使用下列值。 您可以使用這些值來建立測試環境，或參考這些值，進一步了解範例。 如需有關 VPN 閘道的一般設定詳細資訊，請參閱 [關於 VPN 閘道設定](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)。

連線規格：

 - **VPN 類型**：路由型
 - **連線類型**︰站對站 (IPsec)
 - **閘道類型**：VPN
 - **Azure 連線名稱**：Azure-Gateway-AzureStack-S2SGateway (入口網站會自動填入此值)
 - **Azure Stack 連線名稱**：AzureStack-Gateway-Azure-S2SGateway (入口網站會自動填入此值)
 - **共用金鑰**：任何與 VPN 硬體相容、在連線兩端皆具有相符值的金鑰
 - **訂用帳戶**：任何慣用的訂用帳戶
 - **資源群組**：Test-Infra

網路和子網路 IP 位址：

| Azure/Azure Stack 連線 | 名稱 | 子網路 | IP 位址 |
|-------------------------------------|---------------------------------------------|---------------------------------------|-----------------------------|
| Azure vNet | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Azure Stack vNet | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Azure 虛擬網路閘道 | Azure-Gateway |  |  |
| Azure Stack 虛擬網路閘道 | AzureStack-Gateway |  |  |
| Azure 公用 IP | Azure-GatewayPublicIP |  | 在建立時決定 |
| Azure Stack 公用 IP | AzureStack-GatewayPublicIP |  | 在建立時決定 |
| Azure 區域網路閘道 | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Azure Stack 公用 IP 值 |
| Azure Stack 區域網路閘道 | Azure-S2SGateway<br>10.100.102.0/23 |  | Azure 公用 IP 值 |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack"></a>在全域 Azure 與 Azure Stack 中建立虛擬網路

透過 Azure 入口網站，使用下列步驟來建立虛擬網路。 如果您使用本文作為教學課程，則可使用[範例值](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values)。 但是，如果您使用本文來設定生產環境，請以您自己的值取代範例設定。

> [!IMPORTANT]
> 您必須確定在 Azure 或 Azure Stack vNet 位址空間中沒有任何重疊的 IP 位址。

若要在 Azure 中建立 vNet：

1. 使用瀏覽器連線至 [Azure 入口網站](http://portal.azure.com/) ，並使用您的 Azure 帳戶登入。
2. 選取 [建立資源]。 在 [搜尋 Marketplace] 欄位中，輸入 `virtual network`'`。 在結果清單中尋找 [虛擬網路]，然後選取 [虛擬網路]。
3. 從 [選取部署模型] 清單，選取 [Resource Manager]，然後選取 [建立]。
4. 在 [建立虛擬網路] 上進行 VNet 設定。 必填欄位的名稱前面會加上紅色星號。  當您輸入有效值時，星號會變成綠色核取記號。

若要在 Azure Stack 中建立 vNet：

* 使用 Azure Stack 的**租用戶入口網站**，重複上述步驟 (1-4)。

## <a name="add-a-gateway-subnet"></a>新增閘道子網路

將虛擬網路連線到閘道之前，您必須先為您要連線的虛擬網路建立閘道子網路。 閘道服務會使用您在閘道子網路中指定的 IP 位址。

在 [Azure 入口網站](http://portal.azure.com/)中，瀏覽至要建立虛擬網路閘道的 Resource Manager 虛擬網路。

1. 選取 vNet 以開啟 [虛擬網路] 頁面。
2. 在 [設定] 中，選取 [子網路]。
3. 在 [子網路] 頁面中，選取 [+閘道子網路] 以開啟 [新增子網路] 頁面。

    ![新增閘道子網路](media/azure-stack-solution-hybrid-connectivity/image4.png)

4. 子網路的 [名稱] 會自動填入 'GatewaySubnet' 這個值。 若要讓 Azure 將此子網路視為閘道子網路，則必須要有此值。
5. 變更所提供的 [位址範圍] 值，以符合您的組態需求，然後選取 [確定]。

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>在 Azure 與 Azure Stack 中建立虛擬網路閘道

在 Azure 中使用下列步驟來建立虛擬網路閘道。

1. 在入口網站頁面的左側，選取 **+** 並且在搜尋欄位中輸入「虛擬網路閘道」。
2. 在 [結果] 中，選取 [虛擬網路閘道]。
3. 在 [虛擬網路閘道] 中，選取 [建立] 以開啟 [建立虛擬網路閘道] 頁面。
4. 在 [建立虛擬網路閘道] 上，指定網路閘道的值，如 [教學課程範例值] 所示，以及下列其他值：

    - **SKU**：基本
    - **虛擬網路**：選取您先前建立的虛擬網路。 系統會自動選取您建立的閘道子網路。
    - **第一個 IP 組態**：這是您閘道的公用 IP。
        - 選取 [建立閘道 IP 組態]，您即會導向至 [選擇公用 IP 位址] 頁面。
        - 選取 [+新建]，以開啟 [建立公用 IP 位址] 頁面。
        - 輸入公用 IP 位址的 [名稱]。 讓 SKU 保持為 [基本]，然後選取 [確定] 以儲存變更。

       > [!Note]
       > VPN 閘道目前僅支援動態公用 IP 位址配置。 不過，這不表示之後的 IP 位址變更已指派至您的 VPN 閘道。 公用 IP 位址只會在刪除或重新建立閘道時變更。 重新調整、重設或 VPN 閘道的其他內部維護/升級不會變更 IP 位址。

4. 確認您的閘道設定。
5. 選取 [建立] 來建立 VPN 閘道。 閘道設定會經過驗證，而 [部署虛擬網路閘道] 圖格會顯示在儀表板上。

   >[!Note]
   >建立閘道可能需要長達 45 分鐘。 您可能需要重新整理入口網站頁面，才能看到完成的狀態。

    建立閘道之後，您可以在入口網站中查看虛擬網路，檢視已指派給閘道的 IP 位址。 閘道會顯示為已連接的裝置。 如需有關閘道的詳細資訊，請選取裝置。

6. 對您的 Azure Stack 部署重複上述步驟 (1-5)。

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack"></a>在 Azure 與 Azure Stack 中建立區域網路閘道

區域網路閘道通常是指您的內部部署位置。 您會為網站提供 Azure 或 Azure Stack 可以參考的名稱，然後指定：

- 內部部署 VPN 裝置的 IP 位址，而這是您要建立連線的 VPN 裝置。
- IP 位址首碼，以供系統透過 VPN 閘道路由至 VPN 裝置。 您指定的位址首碼是位於內部部署網路上的首碼。

  >[!Note]
  >如果您的內部部署網路變更，或者您需要變更 VPN 裝置的公用 IP 位址，您稍後可以輕鬆地更新這些值。

1. 在入口網站中，選取 [+建立資源]。
2. 在搜尋方塊中輸入**區域網路閘道**，然後選取 **Enter** 鍵進行搜尋。 這將會傳回結果清單。
3. 選取 [區域網路閘道]，然後選取 [建立] 以開啟 [建立區域網路閘道] 頁面。
4. 在 [建立區域網路閘道] 上，使用 [教學課程範例值] 來指定區域網路閘道的值。 包括下列額外值。

    - **IP 位址**：這是您要讓 Azure 或 Azure Stack 連線之 VPN 裝置的公用 IP 位址。 指定不在 NAT 後方的有效公用 IP 位址，以便 Azure 到達此位址。 如果您目前沒有 IP 位址，您可以使用範例中的值作為預留位置，但您必須返回並以您的 VPN 裝置公用 IP 位址取代該預留位置。 在您提供有效的位址前，Azure 無法連線到裝置。
    - **位址空間**：這是此區域網路所代表網路的位址範圍。 您可以加入多個位址空間範圍。 確定您指定的範圍，不會與您要連線的其他網路範圍重疊。 Azure 會將您指定的位址範圍路由傳送至內部部署 VPN 裝置 IP 位址。 如果您想要連線至內部部署網站，請使用您自己的值，而不是範例值。
    - **設定 BGP 設定**：僅在設定 BGP 時才使用。 否則，請勿選取此項目。
    - **訂用帳戶**：確認顯示的是正確的訂用帳戶。
    - **資源群組**：選取要使用的資源群組。 您可以建立新的資源群組，或選取您已經建立的資源群組。
    - **位置**：選取將要建立此物件的位置。 您可以選取 VNet 所在的相同位置，但您不需要這麼做。
5. 當您完成必要值的指定時，請選取 [建立] 以建立區域網路閘道。
6. 對您的 Azure Stack 部署重複這些步驟 (1-5)。

## <a name="configure-your-connection"></a>設定您的連線

內部部署網路的站對站連線需要 VPN 裝置。 您設定的 VPN 裝置稱之為「連線」。 若要設定您的「連線」，您必須：

- 共用金鑰。 這個共同金鑰與您建立站對站 VPN 連線時指定的共用金鑰相同。 在我們的範例中，我們會使用基本的共用金鑰。 我們建議您產生更複雜的金鑰以供使用。
- 虛擬網路閘道的公用 IP 位址。 您可以使用 Azure 入口網站、PowerShell 或 CLI 來檢視公用 IP 位址。 若要使用 Azure 入口網站尋找 VPN 閘道的公用 IP 位址，請瀏覽至 [虛擬網路閘道]，然後選取閘道名稱。

使用下列步驟，在虛擬網路閘道與內部部署 VPN 裝置之間建立站對站 VPN 連線。

1. 在 Azure 入口網站中，選取 [+建立資源]。
2. 搜尋 [連線]。
3. 在 [結果] 中，選取 [連線]。
4. 在 [連線] 上，選取 [建立]。
5. 在 [建立連線] 上，進行下列設定：

    - **連線類型**：選取 [站對站 (IPSec)]。
    - **資源群組**：選取您的測試資源群組。
    - **虛擬網路閘道**：選取您所建立的虛擬網路閘道。
    - **區域網路閘道**：選取您所建立的區域網路閘道。
    - **連線名稱**：這會自動填入來自兩個閘道的值。
    - **共用金鑰**：此值必須與您用於本機內部部署 VPN 裝置的值相符。 教學課程範例會使用 'abc123'，但是您可以(且應該) 使用更為複雜的值。 重要的是，此值必須與您在設定 VPN 裝置時指定的值相同。
    - [訂用帳戶]、[資源群組] 和 [位置] 的值是固定的。

6. 選取 [確定] 來建立連線。

您可以在虛擬網路閘道的 [連線] 頁面中看見此連線。 狀態將會從 [未知] 變成 [連線中]，然後變成 [成功]。

## <a name="next-steps"></a>後續步驟

- 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](https://docs.microsoft.com/azure/architecture/patterns)。
