---
title: Azure VMware 解決方案上的部署範圍
description: 瞭解如何在 Azure VMware 解決方案上部署 VMware 的範圍。
ms.topic: how-to
ms.date: 09/29/2020
ms.openlocfilehash: 2cf6fc5cb7662188650365cb019774d6c778d405
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98684870"
---
# <a name="deploy-horizon-on-azure-vmware-solution"></a>Azure VMware 解決方案上的部署範圍 

>[!NOTE]
>本檔著重于 VMware 的範圍產品，之前稱為「範圍7」。 範圍是與 Azure 上的雲端範圍不同的解決方案，不過有一些共用元件。 Azure VMware 解決方案的主要優點包括更簡單的調整大小方法，以及將 VMware Cloud Foundation 管理整合到 Azure 入口網站中。

[VMware](https://www.vmware.com/products/horizon.html)的®、虛擬桌面和應用程式平臺，在資料中心執行，並提供簡單且集中式的管理。 它可在任何裝置上隨時隨地提供虛擬桌面和應用程式。 範圍可讓您建立和建立與 Windows 和 Linux 虛擬桌面、遠端桌面伺服器 (RDS) 託管應用程式、桌上型電腦和實體機器之間的連線。

在這裡，我們將特別著重于在 Azure VMware 解決方案上部署的範圍。 如需 VMware 的一般資訊，請參閱集中生產檔：

* [VMware 的範圍為何？](https://www.vmware.com/products/horizon.html)

* [深入瞭解 VMware 範圍](https://docs.vmware.com/en/VMware-Horizon/index.html)

* [範圍參考架構](https://techzone.vmware.com/resource/workspace-one-and-horizon-reference-architecture)

有了 Azure VMware 解決方案的範圍簡介，現在有兩個虛擬桌面基礎結構 (VDI) Azure 平臺上的解決方案。 下圖摘要說明高層級的主要差異。

:::image type="content" source="media/horizon/difference-horizon-azure-vmware-solution-horizon-cloud-azure.png" alt-text="Azure 上的 Azure VMware 解決方案和範圍雲端範圍" border="false":::

範圍8版的2006和更新版本支援內部部署和 Azure VMware 解決方案部署。 有一些在內部部署環境中支援但無法在 Azure VMware 解決方案上使用的範圍功能。 也支援範圍生態系統中的其他產品。 如需詳細資訊，請參閱功能同位 [和互通性](https://kb.vmware.com/s/article/80850)。

## <a name="deploy-horizon-in-a-hybrid-cloud"></a>在混合式雲端中部署範圍

當您使用集中式雲端 Pod 架構 (CPA) 來與內部部署和 Azure 資料中心互連時，您可以在混合式雲端環境中部署範圍。 CPA 可擴大您的部署、建立混合式雲端，並供應商務持續性和嚴重損壞修復的冗余。  如需詳細資訊，請參閱 [擴充現有的範圍7環境](https://techzone.vmware.com/resource/business-continuity-vmware-horizon#_Toc41650874)。

>[!IMPORTANT]
>CPA 不是延伸的部署;每個水準 pod 都是相異的，而且所有屬於個別 pod 的連接伺服器都必須位於單一位置，並從網路的觀點來執行于相同的廣播網域上。

就像內部部署或私用資料中心一樣，範圍可以部署在 Azure VMware 解決方案私人雲端中。 在下列各節中，我們將討論在內部部署環境和 Azure VMware 解決方案中部署範圍的主要差異。

Azure 私用雲端在概念上與 VMware SDDC 相同，此詞彙通常用於範圍廣泛的檔。 本檔的其餘部分會使用 Azure 私用雲端和 VMware SDDC 可互換的詞彙。

若要管理訂用帳戶授權，Azure VMware 解決方案上的範圍必須要有範圍雲端連接器。 雲端連接器可以部署在 Azure 虛擬網路中，並與水準連線伺服器一起部署。

>[!IMPORTANT]
>目前尚無法使用 Azure VMware 解決方案範圍的範圍控制平面支援。 請務必下載範圍雲端連接器的 VHD 版本。

## <a name="vcenter-cloud-admin-role"></a>vCenter 雲端系統管理員角色

由於 Azure VMware 解決方案是一個 SDDC 服務，而 Azure 會管理 Azure VMware 解決方案上的 SDDC 生命週期，因此 Azure VMware 解決方案上的 vCenter 許可權模型受限於設計。

客戶必須使用雲端系統管理員角色，此角色具有一組有限的 vCenter 許可權。 您可以使用 Azure VMware 解決方案上的雲端系統管理員角色來修改範圍產品，特別是：

* 立即複製布建已修改為可在 Azure VMware 解決方案上執行。

* 特定的 vSAN 原則 (VMware_Horizon) 是在 Azure VMware 解決方案上建立，以搭配使用，因此必須可供使用，並可用於針對範圍部署的 SDDCs。

* 在 Azure VMware 解決方案上執行時，vSphere 以內容為基礎的讀取快取 (CBRC) （也稱為「查看儲存體加速器」）已停用。

>[!IMPORTANT]
>CBRC 不得重新開啟。

>[!NOTE]
>Azure VMware 解決方案會自動設定特定的範圍設定，只要您部署的範圍是 2006 (，也就是在「範圍8」分支上部署的範圍) ，然後在「水準連線伺服器」安裝程式中選取 [ **Azure** ] 選項。

## <a name="horizon-on-azure-vmware-solution-deployment-architecture"></a>Azure VMware 解決方案部署架構的範圍

一般的範圍架構設計會使用 pod 和封鎖策略。 區塊是單一 vCenter，而多個區塊結合了 pod。 水準 pod 是由水準規模調整限制所決定的組織單位。 每個水準 pod 都有個別的入口網站，因此標準設計作法是將 pod 數目降至最低。

每個雲端都有自己的網路連接配置。 相較于 VMware SDDC 網路/NSX Edge，Azure VMware 解決方案網路連線提供了不同于內部部署之部署範圍的獨特需求。

每個 Azure 私用雲端和 SDDC 都可以處理4000桌面或應用程式會話，假設：

* 工作負載流量會與 LoginVSI 工作背景工作設定檔對齊。

* 只會考慮通訊協定流量，沒有使用者資料。

* NSX Edge 設定為大。

>[!NOTE]
>您的工作負載設定檔和需求可能會不同，因此結果可能會根據您的使用案例而有所不同。 使用者資料量可能會降低工作負載內容中的規模限制。 調整規模，並據以規劃您的部署。 如需詳細資訊，請參閱「 [適用于範圍部署的 Azure VMware 解決方案主機大小](#size-azure-vmware-solution-hosts-for-horizon-deployments) 」一節中的大小調整指導方針。

假設 Azure 私用雲端和 SDDC 的最大限制，我們建議您在 Azure 虛擬網路內執行 (UAGs) 的水準連線伺服器和 VMware 整合存取閘道的部署架構。 它會將每個 Azure 私用雲端和 SDDC 有效地轉換成一個區塊。 然後，將 Azure VMware 解決方案上執行的範圍擴充性最大化。

從 Azure 虛擬網路到 Azure 私用雲端/SDDCs 的連線，應使用 ExpressRoute FastPath 進行設定。 下圖顯示基本的範圍 pod 部署。

:::image type="content" source="media/horizon/horizon-pod-deployment-expresspath-fast-path.png" alt-text="使用 ExpressPath 快速路徑的一般水準 pod 部署" border="false":::

## <a name="network-connectivity-to-scale-horizon-on-azure-vmware-solution"></a>在 Azure VMware 解決方案上調整規模範圍的網路連線能力

本節使用一些常見的部署範例，以協助您在 Azure VMware 解決方案上進行大規模調整，以提供更高層級的網路架構。 重點是專門針對重要的網路元素。 

### <a name="single-horizon-pod-on-azure-vmware-solution"></a>Azure VMware 解決方案上的單一範圍 pod

:::image type="content" source="media/horizon/single-horizon-pod-azure-vmware-solution.png" alt-text="Azure VMware 解決方案上的單一範圍 pod" border="false":::

單一水準 pod 是最直接的部署案例，因為您只會在美國東部區域部署一個範圍的 pod。  由於每個私用雲端和 SDDC 都已預估為處理4000桌面會話，因此您會部署最大範圍的 pod 大小。  您可以規劃最多三個私用雲端/SDDCs 的部署。

透過將基礎結構虛擬機器 (Vm) 部署在 Azure 虛擬網路中，您可以觸達每個水準 pod 的12000會話。 每個私人雲端和 SDDC 與 Azure 虛擬網路之間的連線是 ExpressRoute Fast 路徑。  私用雲端之間不需要任何東西部流量。 

這個基本部署範例的主要假設包括：

* 您沒有想要使用雲端 Pod 架構連接到這個新 pod 的內部部署範圍 pod (CPA) 。

* 終端使用者透過網際網路 (連接到其虛擬桌面，而不是透過內部部署資料中心) 連接。

您可以透過 VPN 或 ExpressRoute 線路，將 Azure 虛擬網路中的 AD 網域控制站連線至內部部署 AD。

基本範例的變化可能是為了支援內部部署資源的連線能力。 例如，使用者可以使用 CPA 存取桌面並產生虛擬桌面應用程式流量，或連接到內部部署的範圍 pod。

此圖顯示如何支援內部部署資源的連線能力。 若要將您的公司網路連線到 Azure 虛擬網路，您需要 ExpressRoute 線路。  您也需要將您的公司網路與每個私用雲端連線，並使用 ExpressRoute 全球接觸來 SDDCs。  它允許從 SDDC 連接到 ExpressRoute 線路和內部部署資源。 

:::image type="content" source="media/horizon/connect-corporate-network-azure-virtual-network.png" alt-text="將您的公司網路連線到 Azure 虛擬網路" border="false":::

### <a name="multiple-horizon-pods-on-azure-vmware-solution-across-multiple-regions"></a>跨多個區域的 Azure VMware 解決方案上有多個範圍的 pod

另一個案例是在多個 pod 之間調整規模。  在此案例中，您會在兩個不同的區域中部署兩個範圍的 pod，並使用 CPA 進行同盟。 它類似于前一個範例中的網路設定，但有一些額外的跨區域連結。 

您會將 Azure 虛擬網路輸入每個區域連線到另一個區域中的私人雲端/SDDCs。 它可讓 CPA 同盟的範圍內連線伺服器連接到管理下的所有桌面。 將其他私人雲端/SDDCs 新增至此設定，可讓您整體調整為24000個會話。 

如果您在相同區域中部署兩個水準 pod，則適用相同的原則。  請務必在 *不同的 Azure 虛擬網路* 中部署第二個範圍的 pod。 就像單一 pod 範例一樣，您可以使用 ExpressRoute 和全球接觸來將您的公司網路和內部部署 pod 連線到此多 pod/區域範例。 

:::image type="content" source="media/horizon/multiple-horizon-pod-azure-vmware-solution.png" alt-text=" 跨多個區域的 Azure VMware 解決方案上有多個範圍的 pod" border="false":::

## <a name="size-azure-vmware-solution-hosts-for-horizon-deployments"></a>適用于範圍部署的 Azure VMware 解決方案主機大小 

在 Azure VMware 解決方案中執行的主機上，範圍的大小調整方法比在內部部署環境中的範圍更簡單。  這是因為 Azure VMware 解決方案主機已標準化。  確切的主機大小有助於判斷支援 VDI 需求所需的主機數目。  它是決定每一桌面成本的核心。

### <a name="sizing-tables"></a>調整資料表大小

範圍虛擬桌面的特定 vCPU/vRAM 需求取決於客戶的特定工作負載設定檔。   請與您的 MSFT 和 VMware sales 小組合作，以協助判斷您的虛擬桌面電腦的 vCPU/vRAM 需求。 

| 每個 VM 的 vCPU | 每個 VM (GB) 的 vRAM | 執行個體 | 100 Vm | 200 Vm | 300 Vm | 400 Vm | 500 Vm | 600 Vm | 700 Vm | 800 Vm | 900 Vm | 1000 Vm | 2000 Vm | 3000 Vm | 4000 Vm | 5000 Vm | 6000 Vm | 6400 Vm |
|:-----------:|:----------------:|:--------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|
|      2      |        3.5       |    AVS   |    3    |    3    |    4    |    4    |    5    |    6    |    6    |    7    |    8    |     9    |    17    |    25    |    33    |    41    |    49    |    53    |
|      2      |         4        |    AVS   |    3    |    3    |    4    |    5    |    6    |    6    |    7    |    8    |    9    |     9    |    18    |    26    |    34    |    42    |    51    |    54    |
|      2      |         6        |    AVS   |    3    |    4    |    5    |    6    |    7    |    9    |    10   |    11   |    12   |    13    |    26    |    38    |    51    |    62    |    75    |    79    |
|      2      |         8        |    AVS   |    3    |    5    |    6    |    8    |    9    |    11   |    12   |    14   |    16   |    18    |    34    |    51    |    67    |    84    |    100   |    106   |
|      2      |        12        |    AVS   |    4    |    6    |    9    |    11   |    13   |    16   |    19   |    21   |    23   |    26    |    51    |    75    |    100   |    124   |    149   |    158   |
|      2      |        16        |    AVS   |    5    |    8    |    11   |    14   |    18   |    21   |    24   |    27   |    30   |    34    |    67    |    100   |    133   |    165   |    198   |    211   |
|      4      |        3.5       |    AVS   |    3    |    3    |    4    |    5    |    6    |    7    |    8    |    9    |    10   |    11    |    22    |    33    |    44    |    55    |    66    |    70    |
|      4      |         4        |    AVS   |    3    |    3    |    4    |    5    |    6    |    7    |    8    |    9    |    10   |    11    |    22    |    33    |    44    |    55    |    66    |    70    |
|      4      |         6        |    AVS   |    3    |    4    |    5    |    6    |    7    |    9    |    10   |    11   |    12   |    13    |    26    |    38    |    51    |    62    |    75    |    79    |
|      4      |         8        |    AVS   |    3    |    5    |    6    |    8    |    9    |    11   |    12   |    14   |    16   |    18    |    34    |    51    |    67    |    84    |    100   |    106   |
|      4      |        12        |    AVS   |    4    |    6    |    9    |    11   |    13   |    16   |    19   |    21   |    23   |    26    |    51    |    75    |    100   |    124   |    149   |    158   |
|      4      |        16        |    AVS   |    5    |    8    |    11   |    14   |    18   |    21   |    24   |    27   |    30   |    34    |    67    |    100   |    133   |    165   |    198   |    211   |
|      6      |        3.5       |    AVS   |    3    |    4    |    5    |    6    |    7    |    9    |    10   |    11   |    13   |    14    |    27    |    41    |    54    |    68    |    81    |    86    |
|      6      |         4        |    AVS   |    3    |    4    |    5    |    6    |    7    |    9    |    10   |    11   |    13   |    14    |    27    |    41    |    54    |    68    |    81    |    86    |
|      6      |         6        |    AVS   |    3    |    4    |    5    |    6    |    7    |    9    |    10   |    11   |    13   |    14    |    27    |    41    |    54    |    68    |    81    |    86    |
|      6      |         8        |    AVS   |    3    |    5    |    6    |    8    |    9    |    11   |    12   |    14   |    16   |    18    |    34    |    51    |    67    |    84    |    100   |    106   |
|      6      |        12        |    AVS   |    4    |    6    |    9    |    11   |    13   |    16   |    19   |    21   |    23   |    26    |    51    |    75    |    100   |    124   |    149   |    158   |
|      6      |        16        |    AVS   |    5    |    8    |    11   |    14   |    18   |    21   |    24   |    27   |    30   |    34    |    67    |    100   |    133   |    165   |    198   |    211   |
|      8      |        3.5       |    AVS   |    3    |    4    |    6    |    7    |    9    |    10   |    12   |    14   |    15   |    17    |    33    |    49    |    66    |    82    |    98    |    105   |
|      8      |         4        |    AVS   |    3    |    4    |    6    |    7    |    9    |    10   |    12   |    14   |    15   |    17    |    33    |    49    |    66    |    82    |    98    |    105   |
|      8      |         6        |    AVS   |    3    |    4    |    6    |    7    |    9    |    10   |    12   |    14   |    15   |    17    |    33    |    49    |    66    |    82    |    98    |    105   |
|      8      |         8        |    AVS   |    3    |    5    |    6    |    8    |    9    |    11   |    12   |    14   |    16   |    18    |    34    |    51    |    67    |    84    |    100   |    106   |
|      8      |        12        |    AVS   |    4    |    6    |    9    |    11   |    13   |    16   |    19   |    21   |    23   |    26    |    51    |    75    |    100   |    124   |    149   |    158   |
|      8      |        16        |    AVS   |    5    |    8    |    11   |    14   |    18   |    21   |    24   |    27   |    30   |    34    |    67    |    100   |    133   |    165   |    198   |    211   |


### <a name="horizon-sizing-inputs"></a>水準調整大小輸入

以下是您所規劃的工作負載需要收集的內容：

* 並行桌面數目

* 每一桌面的必要 vCPU

* 每一桌面的必要 vRAM

* 每個桌面的必要儲存體

一般來說，VDI 部署是 CPU 或 RAM 限制，可決定主機大小。 讓我們使用下列範例來 LoginVSI 知識工作者類型的工作負載，並以效能測試進行驗證：

* 2000並行桌面部署

* 每一桌面2vCPU。

* 每台桌面 vRAM 4 GB。

* 每一桌面 50 GB 的儲存空間

在此範例中，主機的總總數為18，產生每個主機的 VM 每個主機密度111。

>[!IMPORTANT]
>客戶工作負載會與此 LoginVSI 知識工作者的範例不同。 在規劃部署的過程中，請與您的 VMware EUC Se 合作，以滿足您的特定大小和效能需求。 請務必先使用實際的規劃工作負載來執行您自己的效能測試，再完成主機大小調整並據以調整。

## <a name="horizon-on-azure-vmware-solution-licensing"></a>Azure VMware 解決方案授權範圍 

在 Azure VMware 解決方案上執行範圍的整體成本有四個元件。 

### <a name="azure-vmware-solution-capacity-cost"></a>Azure VMware 解決方案容量成本

如需定價的詳細資訊，請參閱 [Azure VMware 解決方案定價](https://azure.microsoft.com/pricing/details/azure-vmware/) 頁面

### <a name="horizon-licensing-cost"></a>範圍授權成本

有兩個適用于 Azure VMware 解決方案的可用授權，可以是並行使用者 (CCU) 或名為使用者 (NU) ：

* 水準訂用帳戶授權

* 範圍通用訂用帳戶授權

如果只部署在 Azure VMware 解決方案的範圍，以供可預見的未來使用，則請使用「水準訂閱」授權，因為這是較低的成本。

如果部署在 Azure VMware 解決方案和內部部署上，如同使用嚴重損壞修復使用案例，請選擇範圍通用訂用帳戶授權。 它包含內部部署的 vSphere 授權，因此成本較高。

請與您的 VMware EUC 銷售小組合作，以根據您的需求判斷範圍授權成本。

### <a name="azure-instance-types"></a>Azure 實例類型

若要瞭解範圍基礎結構所需的 Azure 虛擬機器大小，請參閱可在 [這裡](https://techzone.vmware.com/resource/horizon-on-azure-vmware-solution-configuration#horizon-installation-on-azure-vmware-solution)找到的 VMware 指導方針。

## <a name="next-steps"></a>下一步
若要深入瞭解 Azure VMware 解決方案上的 VMware 範圍，請閱讀 [Vmware 範圍常見問題](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/products/horizon/vmw-horizon-on-microsoft-azure-vmware-solution-faq.pdf)。
