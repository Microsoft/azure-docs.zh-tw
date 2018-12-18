---
title: 如何設定 App Service 環境 v1
description: 設定、管理和監視 App Service 環境 v1
services: app-service
documentationcenter: ''
author: ccompy
manager: stefsch
editor: ''
ms.assetid: b5a1da49-4cab-460d-b5d2-edd086ec32f4
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.openlocfilehash: 34fb3f15c03a3d3ef5f0a27081539bf0a6d19c5f
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22987863"
---
# <a name="configuring-an-app-service-environment-v1"></a>設定 App Service 環境 v1

> [!NOTE]
> 這篇文章是關於 App Service 環境 v1。  有較新版本的 App Service 環境，更易於使用，並且可以在功能更強大的基礎結構上執行。 若要深入了解新版本，請從 [App Service 環境簡介](intro.md)開始。
> 

## <a name="overview"></a>概觀
Azure App Service Environment 是由數個主要元件所組成：

* 在 Azure App Service 環境託管服務中執行的計算資源
* 儲存體
* 資料庫
* 傳統 (V1) 或 Resource Manager(V2) Azure 虛擬網路 (VNet) 
* Azure App Service 環境託管服務執行所在的子網路

### <a name="compute-resources"></a>計算資源
您可以將計算資源用於您的 4 個資源集區。  每個 App Service 環境 (ASE) 都有一組前端和 3 個可能的背景工作集區。 您不需要 3 個背景工作集區都使用--如果您想要，可以只使用其中一或兩個。

租用戶無法直接存取資源集區 (前端和背景工作) 中的主機。 您無法使用遠端桌面通訊協定 (RDP) 來連接它們、變更其佈建，或做為其系統管理員。

但是您可以設定資源集區的數量和大小。 在 ASE 中，您有 4 個大小選項，標記為 P1 到 P4。 如需大小及其價格的詳細資訊，請參閱 [App Service 價格](https://azure.microsoft.com/pricing/details/app-service/)。
變更數量或大小的作業稱為調整作業。  一次只能執行一項調整作業。

**前端**：前端是您保留在 ASE 中之應用程式的 HTTP/HTTPS 端點。 請勿在前端中執行工作負載。

* ASE 是以兩個 P2 開始，這對於開發/測試工作負載及低階生產工作負載而言已經足夠。 強烈建議針對中度到重度生產工作流程使用 P3。
* 對於中度到重度生產工作流程而言，建議至少有 4 個 P3 以確保在排定的維護工作開始執行時有足夠的前端在執行中。 排定的維護活動一次會關閉一個前端。 這會減少維護活動期間整體可用的前端容量。
* 前端最多需要一小時進行佈建。 
* 如需進一步微調，您應監視前端集區的 CPU 使用百分比、記憶體使用百分比，以及使用中要求計量。 如果執行 P3 時 CPU 或記憶體使用百分比超出 70%，請新增更多前端。 如果每個前端的 [使用中要求] 值平均達到 15,000 至 20,000 個要求，您也應該新增更多前端。 整體目標是在執行 P3 時，將 CPU 和記憶體使用百分比維持在低於 70%，以及將每個前端的使用中要求平均數目維持在低於 15,000 個要求。  

**背景工作**：背景工作是您的應用程式實際執行之處。 當您相應增加 App Service 方案時，會用完相關聯背景工作集區中的背景工作。

* 您無法立即新增背景工作。 最多需要一小時進行佈建。
* 調整任何集區的計算資源大小時，每個更新網域都將需要 1 小時來完成。 ASE 中有 20 個更新網域。 如果您調整包含 10 個執行個體之背景工作集區的計算大小，可能需要 10 個小時來完成。
* 如果您變更背景工作集區中使用的計算資源大小，將會導致在該背景工作集區中執行的應用程式冷啟動

變更未執行任何應用程式之背景工作集區計算資源大小的最快方式是：

* 將背景工作角色數量相應減少為 2。  入口網站中相應減少大小最小值為 2。 這將需要幾分鐘時間來解除配置您的執行個體。 
* 選取新的計算大小與執行個體數目。 從這裡開始，將需要 2 小時來完成。

如果您的應用程式需要大型計算資源大小，您就無法利用之前的指引。 您可以改為擴展另一個含有所需大小之背景工作的背景工作集區，並將您的應用程式移到該集區，而不需要變更裝載那些應用程式之背景工作集區的大小。

* 在另一個背景工作集區中建立所需計算大小的其他執行個體。 將需要 1 小時來完成。
* 將您需要較大大小之裝載應用程式的 App Service 方案重新指派至新設定的背景工作集區。 這是一項快速作業，應在一分鐘以內即可完成。  
* 如果您不再需要那些未使用的執行個體，請相應減少第一個背景工作集區。 此作業需要幾分鐘才能完成。

**自動調整**可協助您管理計算資源耗用狀況的其中一項工具。 您可以針對前端或背景工作集區執行自動調整。 您可以執行下列作業：早上增加任何集區類型的執行個體，而晚上減少執行個體。 或者，可以在背景工作集區中可用的背景工作數目低於特定臨界值時，新增執行個體。

如果您想要設定自動調整規則計算資源集區的度量，請記住佈建需要的時間。 如需自動調整 App Service 環境的更多詳細資料，請參閱[如何在 App Service 環境中設定自動調整][ASEAutoscale]。

### <a name="storage"></a>儲存體
每個 ASE 各設有 500 GB 的儲存體。 此空間會用於 ASE 中的所有應用程式。 此儲存空間屬於 ASE 的一部分，且目前無法切換為使用您的儲存空間。 如果您調整虛擬網路路由或安全性，您仍然需要允許存取 Azure 儲存體，否則 ASE 無法運作。

### <a name="database"></a>資料庫
資料庫會保存定義環境的資訊，以及在其中執行之應用程式的詳細資訊。 這也是 Azure 保留的訂用帳戶的一部分。 但是您無法直接加以操作。 如果您調整虛擬網路路由或安全性，您仍然需要允許存取 SQL Azure，否則 ASE 無法運作。

### <a name="network"></a>網路
與 ASE 搭配使用的 VNet，可以是您在建立 ASE 時所設定的虛擬網路，或是您既有的虛擬網路。 當您在 ASE 建立期間建立子網路，它會強制 ASE 位於和虛擬網路相同的資源群組中。 如果您需要 ASE 所使用的資源群組與 VNet 使用的不同，則需要使用 Resource Manager 範本來建立 ASE。

用於 ASE 的虛擬網路有一些限制：

* 虛擬網路必須是區域虛擬網路。
* 需要有包含 8 個或更多位址且為 ASE 部署所在的子網路。
* 將子網路用於裝載 ASE 後，就無法變更子網路的位址範圍。 因此，建議子網路至少包含 64 個位置以適應未來 ASE 成長。
* 除了 ASE 以外，子網路中可能沒有其他項目。

不同於包含 ASE 的託管服務，[虛擬網路][virtualnetwork]和子網路全都在使用者的控制之下。  您可以透過虛擬網路 UI 或 Powershell 管理您的虛擬網路。  ASE 可以部署在傳統或 Resource Manager VNet 中。  傳統和 Resource Manager Vnet 之間的入口網站和 API 經驗會稍有不同，但 ASE 經驗相同。

用來裝載 ASE 的 VNet 可以使用私人 RFC1918 IP 位址或使用公用 IP 位址。  如果您想要使用 RFC1918 (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) 所未涵蓋的 IP 範圍，您必須在建立 ASE 之前，建立 VNet 和子網路以供 ASE 使用。

因為這項功能會將 Azure App Service 放入您的虛擬網路，這表示裝載於 ASE 的應用程式現在可以存取直接透過 ExpressRoute 或網站間虛擬網路 (VPN) 提供的資源。 您的 App Service 環境中的應用程式不需要額外的網路功能，即可存取裝載 App Service 環境的虛擬網路可用的資源。 這表示您不需要使用 VNET 整合或混合式連線，即可將資源移入或連接到您的虛擬網路。 但您仍然可以使用這兩個功能，存取網路中未連接到虛擬網路的資源。

比方說，您可以使用 VNET 整合，與您的訂用帳戶中未連接到 ASE 所在虛擬網路的虛擬網路整合。 您仍可使用混合式連線來存取其他網路中的資源，一如既往。  

如果您的虛擬網路設定了 ExpressRoute VPN，則應留意 ASE 的某些路由需求。 某些使用者定義的路由 (UDR) 組態與 ASE 不相容。 如需關於在具有 ExpressRoute 的虛擬網路中執行 ASE 的詳細資訊，請參閱[在具有 ExpressRoute 的虛擬網路中執行 App Service 環境][ExpressRoute]。

#### <a name="securing-inbound-traffic"></a>保護輸入流量
有兩種主要方法可控制您的 ASE 的輸入流量。  您可以使用網路安全性 (NSG) 來控制哪些 IP 位址可以存取 ASE (請參閱 [如何在 App Service 環境中控制輸入流量](app-service-app-service-environment-control-inbound-traffic.md) )，您也可以使用內部負載平衡器 (ILB) 設定您的 ASE。  如果您要將使用 USG 的存取權限制為您的 ILB ASE，這些功能也可以一起使用。

當您建立 ASE 時，它會在您的 VNet 中建立 VIP。  VIP 類型有兩種：外部和內部。  當您使用外部 VIP 建立 ASE 時，可以透過網際網路可路由 IP 位址存取 ASE 中的應用程式。 如果您選取內部，您的 ASE 會使用 ILB 進行設定，將無法透過網際網路直接存取。  ILB ASE 仍需要外部 VIP，但它只用來進行 Azure 管理和維護。  

在 ILB ASE 建立期間，您會提供 ILB ASE 所用的子網域，而且必須針對您所指定的子網域管理自己的 DNS。  因為您設定了子網域名稱，所以也需要管理用於 HTTPS 存取的憑證。  建立 ASE 之後，系統會提示您提供憑證。  若要深入了解如何建立和使用 ILB ASE，請參閱[在 App Service 環境中使用內部負載平衡器][ILBASE]。 

## <a name="portal"></a>入口網站
您可以使用 Azure 入口網站中的 UI 來管理和監視 App Service 環境。 如果您有 ASE，則可能會在資訊看板上看到 App Service 符號。 這個符號用來代表 Azure 入口網站中的 App Service 環境：

![App Service 環境符號][1]

若要開啟 UI 以列出您所有的 App Service 環境，您可以使用圖示，或選取資訊看板底部的大於箭號 (">" 符號) 來選取 App Service 環境。 選取其中一個列出的 ASE，會開啟用來監視及管理 ASE 的 UI。

![用於監視和管理 App Service 環境的 UI][2]

這是第一個刀鋒視窗，顯示 ASE 的某些屬性和每個資源集區的計量圖表。 某些顯示於 [基本功能] 區塊中的屬性同時也是超連結，將會開啟與其相關聯的刀鋒視窗。 例如，您可以選取**虛擬網路**名稱，以開啟與您的 ASE 執行所在的虛擬網路相關聯的 UI。 **App Service 方案**和**應用程式**分別會開啟不同的刀鋒視窗，列出位於您 ASE 中的項目。  

### <a name="monitoring"></a>監視
圖表可讓您查看每個資源集區中各種不同的效能計量。 對於前端集區，您可以監視平均 CPU 和記憶體使用量。 對於背景工作集區，您可以監視已用和可用的數量。

多個 App Service 方案可能會使用同一個背景工作角色集區中的背景工作。 工作負載並不是透過和前端伺服器相同的方式散佈，所以 CPU 和記憶體使用量不會提供許多有用的資訊。 更重要的是要追蹤您已使用和可用的背景工作角色數目，特別是在您管理此系統以供他人使用時。  

您也可以使用所有可在圖表中追蹤的計量來設定警示。 在此設定警示的效果與在 App Service 中的其他位置相同。 您可以從 [警示] UI 部分設定警示，或深入探索任何計量 UI 並選取 [新增警示] 來設定。

![度量 UI][3]

剛剛所討論的計量是 App Service 環境計量。 另外也有可在 App Service 方案層級上使用的計量。 在其中監視 CPU 和記憶體是很合理的作法。

在 ASE 中，所有 App Service 方案都是專用的 App Service 方案。 這表示，只有在配置給該 App Service 方案的主機上執行的應用程式是該 App Service 方案中的應用程式。 若要查看 App Service 方案的詳細資料，請直接從 ASE UI 中的任何清單或藉由 **瀏覽 App Service 方案** (可列出全部的方案)，來顯示您的 App Service 方案。   

### <a name="settings"></a>設定
ASE 刀鋒視窗中有一個包含幾項重要功能的 [設定]  區段：

[設定] > [屬性]：[設定] 刀鋒視窗會在您顯示 ASE 刀鋒視窗時自動開啟。 最上方是 [屬性] 。 在這裡會看到許多與 [基本功能] 重複的項目，但 [虛擬 IP 位址] 和 [輸出 IP 位址] 是非常好用的項目。

![設定刀鋒視窗和屬性][4]

**設定** > ：當您在 ASE 中建立 IP 安全通訊端層 (SSL) 應用程式時，您需要一個 IP SSL 位址。 若要取得該位址，您的 ASE 需要擁有一些可配置的 IP SSL 位址。 建立 ASE 後，它會有一個 IP SSL 可供此用途使用，但您可以新增更多位址。 額外的 IP SSL 位址需付費，如 [App Service 價格][AppServicePricing] (在 SSL 連線一節中) 所示。 額外的價格是 IP SSL 價格。

[設定] > [前端集區] / [背景工作集區]：這些資源集區刀鋒視窗每一個都可讓您檢視該資源集區的資訊，以及提供充分調整該資源集區所需的控制。  

每個資源集區的基準刀鋒視窗分別會提供一個圖表，內附該資源集區的計量。 就像 ASE 刀鋒視窗中的圖表，您可以進入圖表，並設定所需的警示。 從 ASE 刀鋒視窗為特定的資源集區設定警示，效果等同於從資源集區進行設定。 在背景工作集區的 [設定]  刀鋒視窗中，您可存取在此背景工作集區中執行的所有應用程式或 App Service 方案。

![背景工作集區設定 UI][5]

### <a name="portal-scale-capabilities"></a>入口網站調整功能
共有三種調整作業：

* 變更 ASE 中可供 IP SSL 使用的 IP 位址數目。
* 變更用於資源集區中的計算資源大小。
* 以手動方式或透過自動調整變更用於資源集區中的計算資源數目。

在入口網站中，有三種方式可以控制資源集區中的伺服器數量：

* 從頂端的主要 ASE 刀鋒視窗進行調整作業。 您可以對前端和背景工作集區變更多項調整設定。 它們會以單一作業的方式全部套用。
* 從個別資源集區 [設定] 下的 [調整] 刀鋒視窗手動進行調整作業。
* 從個別資源集區的 [調整]  刀鋒視窗設定的自動調整。

若要使用 ASE 刀鋒視窗上的調整作業，請將滑桿拖曳至所需的數量並儲存。 此 UI 也支援變更大小作業。  

![調整 UI][6]

若要使用特定資源集區中的手動或自動調整估能，請移至適當的 [設定] > [前端集區] / [背景工作集區]。 然後開啟您想要變更的集區。 移至 [設定] > [相應放大] 或 [設定] > [相應增加]。 [相應放大]  刀鋒視窗可讓您控制執行個體數量。  可讓您控制資源大小。  

![調整設定 UI][7]

## <a name="fault-tolerance-considerations"></a>容錯注意事項
您可以將 App Service 環境可設定為最多總計使用 55 個計算資源。 在這 55 個計算資源中，只有 50 個可用來裝載工作負載。 其原因有兩個。 前端計算資源的下限為 2 個。  上限可高達 53 個，以支援背景工作集區配置。 為了提供容錯功能，您需要根據下列規則配置額外的計算資源：

* 每個背景工作角色集區至少需要 1 個無法被指派工作負載的額外計算資源。
* 當背景工作角色集區中的計算資源數量超出特定值時，則需要另一個計算資源以進行容錯。 在前端集區中並非如此。

在任何單一背景工作集區中，容錯需求就是指派給背景工作集區的 X 個資源的指定值：

* 如果 X 介於 2 到 20，您可用於工作負載的可用計算資源數量為 X-1。
* 如果 X 介於 21 到 40，您可用於工作負載的可用計算資源數量為 X-2。
* 如果 X 介於 41 到 53，您可用於工作負載的可用計算資源數量為 X-3。

最少的使用量具有 2 個前端伺服器和 2 個背景工作。  以上陳述可用幾個範例加以釐清：  

* 如果您在單一集區中有 30 個背景工作，則其中有 28 個可用來裝載工作負載。
* 如果您在單一集區中有 2 個背景工作角色，則有 1 個可用來裝載工作負載。
* 如果您在單一集區中有 20 個背景工作角色，則有 19 個可用來裝載工作負載。  
* 如果您在單一集區中有 21 個背景工作角色，則同樣只有 19 個可用來裝載工作負載。  

容錯是很重要的環節，但您在調整超過特定臨界值時必須加以留意。 如果您想要從 20 個執行個體新增更多容量，請調整至 22 或更高，因為 21 並不會額外新增任何容量。 調整為 40 以上時也是一樣，下一個會新增容量的數目為 42。  

## <a name="deleting-an-app-service-environment"></a>刪除 App Service 環境
如果您要刪除 App Service 環境，只需使用 App Service 環境刀鋒視窗頂端的 [刪除]  動作。 當您這麼做時，系統將提示您輸入 App Service 環境的名稱，以確認您真正想要執行這項操作。 請注意，當您刪除 App Service 環境時，將同時刪除其中包含的所有內容。  

![刪除 App Service 環境 UI][9]  

## <a name="getting-started"></a>開始使用
若要開始使用 App Service 環境，請參閱 [如何建立 App Service 環境](app-service-web-how-to-create-an-app-service-environment.md)。

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!--Image references-->
[1]: ./media/app-service-web-configure-an-app-service-environment/ase-icon.png
[2]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-aseblade.png
[3]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolchart.png
[4]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-properties.png
[5]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolblade.png
[6]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-scalecommand.png
[7]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolscale.png
[8]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-pricingtiers.png
[9]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-deletease.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[Appserviceplans]: ../azure-web-sites-web-hosting-plans-in-depth-overview.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[HowtoScale]: app-service-web-scale-a-web-app-in-an-app-service-environment.md
[ControlInbound]: app-service-app-service-environment-control-inbound-traffic.md
[virtualnetwork]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/
[ASEAutoscale]: app-service-environment-auto-scale.md
[ExpressRoute]: app-service-app-service-environment-network-configuration-expressroute.md
[ILBASE]: app-service-environment-with-internal-load-balancer.md
