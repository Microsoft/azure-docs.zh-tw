---
title: 使用 Azure 虛擬 WAN 與 Azure 建立站對站連線 |Microsoft Docs
description: 在本教學課程，您可了解如何使用 Azure 虛擬 WAN 來與 Azure 建立站對站的 VPN 連線。
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 09/21/2018
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to connect my local site to my VNets using Virtual WAN and I don't want to go through a Virtual WAN partner.
ms.openlocfilehash: 20ba28632710ee044d4273ba12900774310711c7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981350"
---
# <a name="tutorial-create-a-site-to-site-connection-using-azure-virtual-wan"></a>教學課程：使用 Azure 虛擬 WAN 來建立站對站連線

本教學課程會示範如何使用虛擬 WAN，透過 IPsec/IKE (IKEv1 和 IKEv2) VPN 連線來與 Azure 中的資源連線。 此類型的連線需要位於內部部署的 VPN 裝置，且您已對該裝置指派對外開放的公用 IP 位址。 如需有關虛擬 WAN 的詳細資訊，請參閱[虛擬 WAN 概觀](virtual-wan-about.md)

> [!NOTE]
> 如果您有許多網站，通常會使用[虛擬 WAN 夥伴](https://aka.ms/virtualwan)來建立此組態。 不過，如果您慣用網路並且知道如何設定自己的 VPN 裝置，可以自行建立此組態。
>

![虛擬 WAN 的圖表](./media/virtual-wan-about/virtualwan.png)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 WAN
> * 建立網站
> * 建立中樞
> * 將中樞連線至網站
> * 將 VNet 連線至中樞
> * 下載並套用 VPN 裝置組態
> * 檢視您的虛擬 WAN
> * 檢視資源健康情況
> * 監視連線

## <a name="before-you-begin"></a>開始之前

[!INCLUDE [Before you begin](../../includes/virtual-wan-tutorial-vwan-before-include.md)]

## <a name="vnet"></a>1.建立虛擬網路

[!INCLUDE [Create a virtual network](../../includes/virtual-wan-tutorial-vnet-include.md)]

## <a name="openvwan"></a>2.建立虛擬 WAN

透過瀏覽器瀏覽至 [Azure 入口網站](https://portal.azure.com) ，並使用您的 Azure 帳戶登入。

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-vwan-include.md)]

## <a name="site"></a>3.建立網站

建立需要的網站數目，這會與您的實體位置相對應。 例如，如果您在紐約 (NY)、倫敦和洛杉磯 (LA) 各有分公司，就會建立三個不同網站。 這些網站包含您內部部署的 VPN 裝置端點。 在此階段中，您可以只為網站指定一個私人位址空間。

1. 按一下您所建立的 WAN。 在 WAN 頁面的 [WAN 架構] 底下，按一下 [VPN 網站] 以開啟 [VPN 網站] 頁面。
2. 在 [VPN 網站] 頁面上，按一下 [+ 建立網站]。
3. 在 [建立網站] 頁面上，填寫下列欄位：

  * **名稱** - 這是您要用來參考內部部署網站的名稱。
  * **公用 IP 位址** - 這是您內部部署網路中 VPN 裝置的公用 IP 位址。
  * **私人位址空間** - 這是位於內部部署網站上的 IP 位址空間。 流向此位址空間的流量會路由至您本機網站。
  * **訂用帳戶** - 請確認訂用帳戶。
  * **資源群組** - 要使用的資源群組。
  * **位置**。
4. 按一下 [進階顯示] 即可檢視其他設定。 您可以選取 **BGP** 以啟用 BGP (會在 Azure 中針對為此網站所建立的所有連線啟用此功能)。 您也可以輸入 [裝置資訊] (選擇性欄位)。 這可協助 Azure 小組深入了解您的環境，以便在未來增加額外最佳化的可能性，或協助您進行疑難排解。
5. 按一下 [確認]。
6. 按一下 [確認] 之後，檢視 [VPN 網站] 頁面上的狀態。 網站會從 [正在佈建] 變成 [已佈建]。

## <a name="hub"></a>4.建立中樞

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-hub-include.md)]

## <a name="associate"></a>5.將網站與中樞相關聯

中樞通常應關聯到 VNet 所在相同區域中的網站。

1. 在 [VPN 網站] 頁面上，選取您想要與中樞相關聯的一或多個網站，然後按一下 [+ 新增中樞關聯]。
2. 在 [將網站與一或多個中樞相關聯] 頁面上，從下拉式清單中選取中樞。 您可以按一下 [+ 新增關聯] 將網站與其他中樞相關聯。
3. 您也可以在這裡新增特定 **PSK**，或是使用預設值。
4. 按一下 [確認]。
5. 您可以在 [VPN 網站] 頁面上檢視連線狀態。

## <a name="vnet"></a>6.將 VNet 連線至中樞

在此步驟中，您會在中樞和 VNet 之間建立等互連的連線。 為您想要連線的每個 VNet 重複這些步驟。

1. 在虛擬 WAN 頁面上，按一下 [虛擬網路連線]。
2. 在 [虛擬網路連線] 頁面上，按一下 [+ 新增連線]。
3. 在 [新增連線] 頁面上，填寫下列欄位︰

    * **名稱** - 為您的連線命名。
    * **中樞** - 選取要與此連線產生關聯的中樞。
    * **訂用帳戶** - 請確認訂用帳戶。
    * **虛擬網路** - 選取要與此中樞連線的虛擬網路。 虛擬網路不能有現有的虛擬網路閘道。
4. 按一下 [確定] 來建立對等互連連線。

## <a name="device"></a>7.下載 VPN 組態

若要設定您的內部部署 VPN 裝置，請使用 VPN 裝置組態。

1. 在您的虛擬 WAN 頁面上，按一下 [概觀]。
2. 在 [概觀] 頁面頂端，按一下 [下載 VPN 組態]。 Azure 會在資源群組 'microsoft-network-[location]' 中建立儲存體帳戶，其中 location 是 WAN 的位置。 將組態套用至您的 VPN 裝置之後，就可以刪除此儲存體帳戶。
3. 檔案建立完成之後，您可以按一下連結來下載。
4. 將組態套用至您的 VPN 裝置。

### <a name="understanding-the-vpn-device-configuration-file"></a>了解 VPN 裝置組態檔

裝置組態檔包含設定內部部署 VPN 裝置時要使用的設定。 當您檢視此檔案時，請注意下列資訊：

* **vpnSiteConfiguration -** 此區段表示網站連線至虛擬 WAN 時設定的裝置詳細資料。 其中包含分支裝置的名稱和公用 IP 位址。
* **vpnSiteConnections -** 本區段提供下列相關資訊：

    * 虛擬中樞 VNet 的**位址空間**<br>範例：
 
        ```
        "AddressSpace":"10.1.0.0/24"
        ```
    * 連線至中樞的 Vnet **位址空間**<br>範例：

         ```
        "ConnectedSubnets":["10.2.0.0/16","10.30.0.0/16"]
         ```
    * 虛擬中樞 Vpngateway 的 **IP 位址**。 由於 Vpngateway 的每個連線都由「主動對主動」設定中的 2 個通道組成，因此您會看到此檔案中列出這兩個 IP 位址。 在此範例中，您會看到每個網站的 "Instance0" 和 "Instance1"。<br>範例：

        ``` 
        "Instance0":"104.45.18.186"
        "Instance1":"104.45.13.195"
        ```
    * **Vpngateway 連線設定詳細資料**，例如 BGP、預先共用金鑰等等。PSK 是自動為您產生的預先共用金鑰。 您隨時都可以在自訂 PSK 的 [概觀] 頁面中編輯連線。
  
### <a name="example-device-configuration-file"></a>範例裝置組態檔

  ```
  { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"r403583d-9c82-4cb8-8570-1cbbcd9983b5"
      },
      "vpnSiteConfiguration":{ 
         "Name":"testsite1",
         "IPAddress":"73.239.3.208"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe",
               "ConnectedSubnets":[ 
                  "10.2.0.0/16",
                  "10.30.0.0/16"
               ]
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.186",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"bkOWe5dPPqkx0DfFE3tyuP7y3oYqAEbI",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   },
   { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"1f33f891-e1ab-42b8-8d8c-c024d337bcac"
      },
      "vpnSiteConfiguration":{ 
         "Name":" testsite2",
         "IPAddress":"66.193.205.122"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe"
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.187",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"XzODPyAYQqFs4ai9WzrJour0qLzeg7Qg",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   },
   { 
      "configurationVersion":{ 
         "LastUpdatedTime":"2018-07-03T18:29:49.8405161Z",
         "Version":"cd1e4a23-96bd-43a9-93b5-b51c2a945c7"
      },
      "vpnSiteConfiguration":{ 
         "Name":" testsite3",
         "IPAddress":"182.71.123.228"
      },
      "vpnSiteConnections":[ 
         { 
            "hubConfiguration":{ 
               "AddressSpace":"10.1.0.0/24",
               "Region":"West Europe"
            },
            "gatewayConfiguration":{ 
               "IpAddresses":{ 
                  "Instance0":"104.45.18.187",
                  "Instance1":"104.45.13.195"
               }
            },
            "connectionConfiguration":{ 
               "IsBgpEnabled":false,
               "PSK":"YLkSdSYd4wjjEThR3aIxaXaqNdxUwSo9",
               "IPsecParameters":{ 
                  "SADataSizeInKilobytes":102400000,
                  "SALifeTimeInSeconds":3600
               }
            }
         }
      ]
   }
  ```

### <a name="configuring-your-vpn-device"></a>設定 VPN 裝置

>[!NOTE]
> 如果您正在處理虛擬 WAN 合作夥伴解決方案，系統會自動進行 VPN 裝置設定。 裝置控制器會從 Azure 取得設定檔，並套用到裝置以設定 Azure 的連線。 這表示您無須知道如何手動設定 VPN 裝置。
>

如果需要設定裝置的指示，您可以使用 [VPN 裝置組態指令碼頁面](~/articles/vpn-gateway/vpn-gateway-about-vpn-devices.md#configscripts)中的指示，並留意下列注意事項：

* VPN 裝置頁面上的指示不是針對虛擬 WAN 所撰寫，但您也可以從組態檔使用虛擬 WAN 的值，手動設定 VPN 裝置。 
* 適用於 VPN 閘道的可下載裝置組態指令碼不適用於虛擬 WAN，因為組態不同。
* 新的虛擬 WAN 可以支援 IKEv1 和 IKEv2。
* 虛擬 WAN 只能使用路由式 VPN 裝置和裝置指示。

## <a name="viewwan"></a>8.檢視您的虛擬 WAN

1. 瀏覽至虛擬 WAN。
2. 在 [概觀] 頁面中，地圖上的每個點都代表著中樞。 暫留在任一點上，即可檢視中樞健康情況摘要。
3. 在 [中樞與連線] 區段中，您可以檢視中樞狀態、網站、區域、VPN 連線狀態和輸入與輸出位元組。

## <a name="viewhealth"></a>9.檢視資源健康情況

1. 瀏覽至您的 WAN。
2. 在 WAN 頁面上的 [支援 + 疑難排解] 區段中，按一下 [健康情況] 並檢視您的資源。

## <a name="connectmon"></a>10.監視連線

建立連線以監視 Azure VM 和遠端站台之間的通訊。 如需有關如何設定連線監視的資訊，請參閱[監視網路通訊](~/articles/network-watcher/connection-monitor.md)。 來源欄位是 Azure 中的 VM IP，目的地 IP 是位址是網站 IP。

## <a name="cleanup"></a>11.清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源。 將 "myResourceGroup" 取代為您的資源群組名稱，然後執行下列 PowerShell 命令：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立 WAN
> * 建立網站
> * 建立中樞
> * 將中樞連線至網站
> * 將 VNet 連線至中樞
> * 下載並套用 VPN 裝置組態
> * 檢視您的虛擬 WAN
> * 檢視資源健康情況
> * 監視連線

若要深入了解虛擬 WAN，請參閱[虛擬 WAN 概觀](virtual-wan-about.md)頁面。
