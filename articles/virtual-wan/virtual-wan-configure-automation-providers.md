---
title: Azure 虛擬 WAN 夥伴 | Microsoft Docs
description: 本文可協助夥伴設定 Azure 虛擬 WAN 自動化。
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/23/2018
ms.author: cherylmc
Customer intent: As a Virtual WAN software-defined connectivity provider, I want to set up a provisioning environment.
ms.openlocfilehash: 7f70470880845fd4271ffdbb35af771ec433babc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961938"
---
# <a name="virtual-wan-partners-preview"></a>虛擬 WAN 夥伴 (預覽)

本文協助您了解如何設定自動化環境，以連線和設定適用於 Azure 虛擬 WAN 的分支裝置 (客戶內部部署 VPN 裝置或 SDWAN CPE)。 如果您是提供透過 IPsec/IKEv2 或 IPsec/IKEv1 使用 VPN 連線的分支裝置提供者，則適合閱讀本文。

分支裝置 (客戶內部部署 VPN 裝置或 SDWAN CPE) 通常會使用佈建的控制器/裝置儀表板。 SD-WAN 解決方案系統管理員通常可使用管理主控台預先佈建裝置，然後才將它插入到網路。 這個具有 VPN 功能的裝置會從控制器取得其控制平面邏輯。 VPN 裝置或 SD-WAN 控制器可以使用 Azure API，將 Azure 虛擬 WAN 的連線自動化。 此類型的連線需要內部部署的裝置，且已對裝置指派對外開放的公用 IP 位址。

## <a name ="before"></a>開始自動化之前

* 請確認您的裝置支援 IPsec IKEv1/IKEv2。 請參閱[預設原則](#default)。
* 請參閱 [REST API](https://docs.microsoft.com/rest/api/azure/)，您將使用它來自動化 Azure 虛擬 WAN 的連線。
* 測試 Azure 虛擬 WAN 的入口網站體驗。
* 然後，決定要自動化哪些建立連線的步驟。 至少，我們建議自動化以下項目：

  * 存取控制
  * 將分支裝置資訊上傳到 Azure 虛擬 WAN
  * 下載 Azure 設定，並設定從分支裝置到 Azure 虛擬 WAN 的連線

* 了解搭配 Azure 虛擬 WAN 可能產生的客戶體驗。

  1. 一般來說，虛擬 WAN 使用者將從建立虛擬 WAN 資源開始這個程序。
  2. 使用者會為內部部署系統 (您的分支控制器或 VPN 裝置佈建軟體) 設定以服務主體為基礎的資源群組，以將分支資訊寫入 Azure 虛擬 WAN。
  3. 使用者可在此時決定登入您的 UI 和設定服務主體認證。 完成後，控制器應該就能透過您將提供的自動化來上傳分支資訊。 在 Azure 端與此對等的手動操作是「建立網站」。
  4. 一旦 Azure 可以提供網站 (分支裝置) 資訊之後，使用者即可將網站關聯到中樞。 虛擬中樞是受 Microsoft 管理的虛擬網路。 中樞包含不同的服務端點，可啟用您內部部署網路 (vpnsite) 中的連線。 中樞是您在區域中的網路核心。 每個 Azure 區域只能有一個中樞，它內部的 VPN 端點 (vpngateway) 會在這個程序期間建立。 VPN 閘道是可調整規模的閘道，其大小是根據頻寬和連線需求來調整。 您可以選擇從分支裝置控制器儀表板自動化虛擬中樞和 vpngateway 的建立程序。
  5. 虛擬中樞與網站相關聯之後，會為使用者產生一個設定檔，供他們手動下載。 此時自動化程序即可發揮作用，讓使用者的體驗變得順暢。 使用者不用手動下載和設定分支裝置，而是由您設定自動化程序，在 UI 上為他們提供最少步驟的點選體驗，因此減少了像是共用金鑰不相符、IPSec 參數不相符、設定檔可讀性等等這些常見的連線問題。
  6. 解決方案中這個步驟接近完成時，使用者將會擁有在分支裝置與虛擬中樞順暢建立站對站連線的體驗。 您也可以對其他中樞設定額外的連線。 每個連線是主動-主動通道。 您的客戶可以選擇針對每個通道的連結使用不同的 ISP。

## <a name ="understand"></a>了解自動化詳細資料


###  <a name="access"></a>存取控制

客戶必須能夠在裝置 UI 中針對虛擬 WAN 設定適當的存取控制。 建議使用 Azure 服務主體進行此設定。 服務主體型存取為裝置控制器提供適當的驗證，以上傳分支資訊。 如需詳細資訊，請參閱[建立服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)。 雖然這項功能不包含在 Azure 虛擬 WAN 產品中，但是我們下面列出在 Azure 中設定存取權所需採取的一般步驟，之後相關的詳細資料會放入裝置管理儀表板

* 為內部部署裝置控制器建立 Azure Active Directory 應用程式。
* 取得應用程式識別碼和驗證金鑰
* 取得租用戶識別碼
* 將應用程式指派給「參與者」角色

###  <a name="branch"></a>上傳分支裝置資訊

設計上傳分支 (內部部署網站) 資訊到 Azure 的使用者體驗。 VPNSite 的 [REST API](https://docs.microsoft.com/rest/api/virtualwan/vpnsites) 可以用於在虛擬 WAN 中建立網站資訊。 您可以提供所有分支 SDWAN/VPN 裝置，或選取適當的裝置自訂。


### <a name="device"></a>裝置設定下載和連線

這個步驟涉及下載 Azure 設定，並設定從分支裝置到 Azure 虛擬 WAN 的連線。 在此步驟中，未使用提供者的客戶將手動下載 Azure 設定，並將之套用到內部部署的 SDWAN/VPN 裝置。 身為提供者，您應該將這個步驟自動化。 裝置控制器可以呼叫 'GetVpnConfiguration' REST API 來下載看起來類似下列檔案的 Azure 設定。

**設定注意事項**

  * 如果 Azure VNet 是附加到虛擬中樞，便會顯示為 ConnectedSubnets。
  * VPN 連線使用路由型設定和 IKEv2/IKEv1。

#### <a name="understanding-the-device-configuration-file"></a>了解裝置設定檔

裝置設定檔包含了設定內部部署 VPN 裝置時所要使用的設定。 當您檢視此檔案時，請注意下列資訊：

* **vpnSiteConfiguration -** 此區段表示網站連線至虛擬 WAN 時設定的裝置詳細資料。 其中包含分支裝置的名稱和公用 IP 位址。
* **vpnSiteConnections -** 此區段提供下列相關資訊：

    * 虛擬中樞 VNet 的**位址空間**。<br>範例：
 
        ```
        "AddressSpace":"10.1.0.0/24"
        ```
    * 連線至中樞之 VNet 的**位址空間**。<br>範例：

         ```
        "ConnectedSubnets":["10.2.0.0/16","10.30.0.0/16"]
         ```
    * 虛擬中樞 Vpngateway 的 **IP 位址**。 由於 Vpngateway 的每個連線都由「主動對主動」設定中的 2 個通道組成，因此您會看到此檔案中列出這兩個 IP 位址。 在此範例中，您會看到每個網站的 "Instance0" 和 "Instance1"。<br>範例：

        ``` 
        "Instance0":"104.45.18.186"
        "Instance1":"104.45.13.195"
        ```
    * **Vpngateway 連線設定詳細資料**，例如 BGP、預先共用金鑰等等。PSK 是自動為您產生的預先共用金鑰。 您隨時都可以在自訂 PSK 的 [概觀] 頁面中編輯連線。
  
#### <a name="example-device-configuration-file"></a>範例裝置設定檔

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

## <a name="default"></a>IPsec 連線的預設原則

### <a name="initiator"></a>啟動器

以下小節列出當 Azure 是通道的啟動器時，所支援的原則組合。

**第 1 階段**

* AES_256、SHA1、DH_GROUP_2
* AES_256、SHA_256、DH_GROUP_2
* AES_128、SHA1、DH_GROUP_2
* AES_128、SHA_256、DH_GROUP_2
* 3DES、SHA1、DH_GROUP_2
* 3DES、SHA_256、DH_GROUP_2

**第 2 階段**

* GCM_AES_256、GCM_AES_256、PFS_NONE
* AES_256、SHA_1、PFS_NONE
* CBC_3DES、SHA_1、PFS_NONE
* AES_256、SHA_256、PFS_NONE
* AES_128、SHA_1、PFS_NONE
* CBC_3DES、SHA_256、PFS_NONE

### <a name="responder"></a>回應者

以下小節列出當 Azure 是通道的回應者時，所支援的原則組合。

**第 1 階段**

* AES_256、SHA1、DH_GROUP_2
* AES_256、SHA_256、DH_GROUP_2
* AES_128、SHA1、DH_GROUP_2
* AES_128、SHA_256、DH_GROUP_2
* 3DES、SHA1、DH_GROUP_2
* 3DES、SHA_256、DH_GROUP_2

**第 2 階段**

* GCM_AES_256、GCM_AES_256、PFS_NONE
* AES_256、SHA_1、PFS_NONE
* CBC_3DES、SHA_1、PFS_NONE
* AES_256、SHA_256、PFS_NONE
* AES_128、SHA_1、PFS_NONE
* CBC_3DES、SHA_256、PFS_NONE
* CBC_DES、SHA_1、PFS_NONE 
* AES_256、SHA_1、PFS_1
* AES_256、SHA_1、PFS_2
* AES_256、SHA_1、PFS_14
* AES_128、SHA_1、PFS_1
* AES_128、SHA_1、PFS_2
* AES_128、SHA_1、PFS_14
* CBC_3DES、SHA_1、PFS_1
* CBC_3DES、SHA_1、PFS_2
* CBC_3DES、SHA_256、PFS_2
* AES_256、SHA_256、PFS_1
* AES_256、SHA_256、PFS_2
* AES_256、SHA_256、PFS_14
* AES_256、SHA_1、PFS_24
* AES_256、SHA_256、PFS_24
* AES_128、SHA_256、PFS_NONE
* AES_128、SHA_256、PFS_1
* AES_128、SHA_256、PFS_2
* AES_128、SHA_256、PFS_14
* CBC_3DES、SHA_1、PFS_14

### <a name="does-everything-need-to-match-between-the-virtual-hub-vpngateway-policy-and-my-on-premises-sdwanvpn-device-or-sd-wan-configuration"></a>虛擬中樞 vpngateway 原則和我的內部部署 SDWAN/VPN 裝置或 SD-WAN 設定之間的所有項目都必須相符嗎？

您的內部部署 SDWAN/VPN 裝置或 SD-WAN 設定必須符合或包含以下您在 Azure IPsec/IKE 原則中指定的演算法和參數。

* IKE 加密演算法
* IKE 完整性演算法
* DH 群組
* IPsec 加密演算法
* IPsec 完整性演算法
* PFS 群組

## <a name="next-steps"></a>後續步驟

如需虛擬 WAN 的詳細資訊，請參閱[關於 Azure 虛擬 WAN](virtual-wan-about.md) 和 [Azure 虛擬 WAN 常見問題集](virtual-wan-faq.md)。

如需任何其他資訊，請傳送電子郵件給 <azurevirtualwan@microsoft.com>。 請在主旨列中以 "[]" 包含您的公司名稱。
