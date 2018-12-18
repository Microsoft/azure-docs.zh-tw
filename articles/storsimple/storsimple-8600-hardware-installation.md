---
title: 安裝 Microsoft Azure StorSimple 8600 裝置 | Microsoft Docs
description: 描述如何打開包裝、掛接機架和佈線 StorSimple 8600 裝置，再部署和設定軟體。
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: 3d82ba5f-3e34-40dc-9c33-50f952bc6be8
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: 5a8b460441323cb668a3d9939cce434636afc44d
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38719000"
---
# <a name="unpack-rack-mount-and-cable-your-storsimple-8600-device"></a>打開包裝、掛接機架和佈線 StorSimple 8600 裝置
## <a name="overview"></a>概觀
您的 Microsoft Azure StorSimple 8600 是雙重機箱裝置，包含主要及 EBOD 機箱。 本教學課程說明如何在您設定 StorSimple 軟體之前，打開包裝、利用機架掛接和配接 StorSimple 8600 裝置硬體纜線。

## <a name="unpack-your-storsimple-8600-device"></a>打開您的 StorSimple 8600 裝置包裝
下列步驟提供如何打開 StorSimple 8600 儲存體裝置包裝的清楚、詳細指示。 這個裝置以兩個箱子出貨，一個用於主要機箱，另一個用於 EBOD 機箱。 然後這兩個箱子會放在單一的箱子中。

### <a name="prepare-to-unpack-your-device"></a>準備打開裝置包裝
打開裝置包裝之前，請檢閱下列資訊。

![警告圖示](./media/storsimple-safety/IC740879.png)![重量圖示](./media/storsimple-8600-hardware-installation/HCS_HeavyWeight_Icon.png) **警告！**

1. 如果您手動處理它，請確定您有兩名人員可以應付裝置的重量。 完全設定的機箱可以重達 32 公斤 (70 磅)。
2. 將箱子放置在平坦的表面上。

接下來，完成下列步驟以打開裝置的包裝。

#### <a name="to-unpack-your-device"></a>打開裝置包裝
1. 檢查箱子及包裝發泡材料有無損毀、割痕、浸水，或任何其他明顯損傷。 如果箱子或包裝嚴重損毀，不要打開箱子。 請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以協助您評估裝置是否可以正常運作。
2. 開啟外箱，然後取出對應主要機箱和 EBOD 機箱的兩個箱子。 您現在可以拆封主要機箱和 EBOD 機箱。 下圖顯示其中一個機箱拆封後的樣子。
   
    ![打開您的儲存體裝置包裝](./media/storsimple-8600-hardware-installation/HCSUnpackyour4Udevice.png)
   
    **儲存體裝置打開包裝的樣子**
   
   | 標籤 | 說明 |
   | --- | --- |
   |   1 |包裝箱 |
   |   2 |SAS 纜線 (在附件和纜線匣) |
   |   3 |底層發泡材料 |
   |   4 |裝置 |
   |   5 |頂層發泡材料 |
   |   6 |附件箱 |
3. 打開兩個箱子的包裝之後，請確定您有：
   
   * 1 個主要機箱 (主要機箱與 EBOD 機箱是在兩個不同的箱子中)
   * 1 個 EBOD 機箱
   * 4 條電源線，每個箱子各 2 條
   * 2 條 SAS 纜線 (連接主要機箱與 EBOD 機箱)
   * 1 條轉接乙太網路纜線
   * 2 條序列主控台纜線
   * 1 個用於序列存取的序列-USB 轉換器
   * 4 個 QSFP-to-SFP+ 配接器以用於 10 GbE 網路介面
   * 2 個機架掛接套件 (4 個側軌掛接硬體，主要機箱與 EBOD 機箱各 2 個)，每個箱子中各 1 個
   * 開始使用文件
     
     如果您未收到任何上述項目，請[連絡 Microsoft 支援](storsimple-8000-contact-microsoft-support.md)。  

下一步是利用機架掛接裝置。

## <a name="rack-mount-your-storsimple-8600-device"></a>機架掛接您的 StorSimple 8600 裝置
遵循下列步驟，以具有前後端柱子的標準 19 英吋機架安裝 StorSimple 8600 儲存體裝置。 此裝置有兩個機箱：主要機箱和 EBOD 機箱。 兩個機箱都需要機架掛接。

安裝包含多個步驟，會在下列程序中討論每個步驟。

> [!IMPORTANT]
> StorSimple 裝置需要機架掛接才能正常運作。
> 
> 

### <a name="site-preparation"></a>場地準備
機箱必須安裝在具有前後端柱子的標準 19 英吋機架上。 使用下列程序來準備機架安裝。

#### <a name="to-prepare-the-site-for-rack-installation"></a>準備場地進行機架安裝
1. 請確定主要和 EBOD 機箱安全地放在平坦、穩定的工作介面 (或類似)。
2. 請確認您想要安裝的場地具有獨立來源的標準 AC 電源，或是具有不斷電供應系統 (UPS) 的機架電源分配單元 (PDU)。
3. 請確定您要掛接機箱的機架上有一個 4U (2 X 2U) 插槽可用。

![警告圖示](./media/storsimple-safety/IC740879.png)![重量圖示](./media/storsimple-8600-hardware-installation/HCS_HeavyWeight_Icon.png) **警告！**

 如果您手動處理裝置安裝，請確定您有兩名人員可以應付裝置的重量。 完全設定的機箱可以重達 32 公斤 (70 磅)。

### <a name="rack-prerequisites"></a>機架必要條件
機箱是針對安裝在具有下列項目的標準 19 英吋機櫃而設計的：

* 從機架柱到柱子最小深度為 27.84 英吋
* 裝置的最大重量為 32 公斤
* 最大背部壓力為 5 Pascal (0.5 mm 水位表)

### <a name="rack-mounting-rail-kit"></a>機架掛接滑軌套件
提供一組掛接滑軌以用於 19 英吋機櫃。 滑軌已經過測試可以處理最大機箱重量。 這些滑軌也可以進行多個機箱的安裝，而不會損失機櫃內的空間。 先安裝 EBOD 機箱。

#### <a name="to-install-the-ebod-enclosure-on-the-rails"></a>在滑軌上安裝 EBOD 機箱
1. 只有在內部滑軌未安裝在您的裝置上時才執行此步驟。 通常，內部滑軌會在工廠安裝。 如果滑軌沒有安裝的話，則在機箱底座側邊安裝左邊和右邊滑軌。 它們是在每一邊使用六個公制螺絲來連接。 為了協助辨識方向，滑軌標示為 [LH – Front] \(左邊 – 前) 和 [RH – Front] \(右邊 – 前)，接至機箱後端的尾端有錐型結尾。
   
    ![將滑軌連接至機箱底座](./media/storsimple-8600-hardware-installation/HCSAttachingRailSlidestoEnclosureChassis.png)
   
    **將滑軌連接至機箱側邊**
   
   | 標籤 | 說明 |
   | --- | --- |
   |  1 |M 3x4 圓頭螺釘 |
   |  2 |底座滑軌 |
2. 將左邊滑軌和右邊滑軌組件連接至機櫃垂直面。 托架會標示 [LH] \(左邊)，[RH] \(右邊) 和 [This side up] \(此面向上)，引導您正確的方向。
3. 找出滑軌組件前後方的滑軌插梢。 延伸滑軌以適合機架柱，並且將插梢插入前後端機架柱垂直面的孔洞。 請確定滑軌組件是水平的。
4. 使用兩個提供的公制螺絲將滑軌組件鎖固至機架垂直面。 在前端和後端各使用一個螺絲。
5. 對其他滑軌組件重複這些步驟。
   
     ![將滑軌連接至機櫃](./media/storsimple-8600-hardware-installation/HCSAttachingRailSlidestoRackCabinet.png)
   
    **將滑軌組件連接至機架**
   
   | 標籤 | 說明 |
   | --- | --- |
   |   1 |固定螺絲 |
   |   2 |方孔前端機架柱螺絲 |
   |   3 |左邊前端滑軌位置插梢 |
   |   4 |固定螺絲 |
   |   5 |左邊後端滑軌位置插梢 |

### <a name="mounting-the-ebod-enclosure-in-the-rack"></a>在機架中掛接 EBOD 機箱
使用剛才安裝的機架滑軌，執行下列步驟以在機架中掛接 EBOD 機箱。

#### <a name="to-mount-the-ebod-enclosure"></a>掛接 EBOD 機箱
1. 使用輔助工具，將機箱抬起並且對齊機架滑軌。
2. 仔細地將機箱插入滑軌，然後將機箱完全推入至機櫃。
   
    ![在機架中插入裝置](./media/storsimple-8600-hardware-installation/HCSInsertingDeviceintheRack.png)
   
    **在機架中掛接機箱**
3. 鬆開蓋子以移除左右前端輪緣蓋。 輪緣蓋只是卡在輪緣上。
4. 藉由在每個輪緣、左側和右側，安裝一個提供的十字螺絲，在機架中鎖固機箱。
5. 安裝輪緣蓋，方法是將其推入定位並且扣住。
   
     ![安裝輪緣蓋](./media/storsimple-8600-hardware-installation/HCSInstallingFlangeCaps.png)
   
    **安裝輪緣蓋**
   
   | 標籤 | 說明 |
   | --- | --- |
   |   1 |機箱鎖固螺絲 |

### <a name="mounting-the-primary-enclosure-in-the-rack"></a>在機架中掛接主要機箱
完成掛接 EBOD 機箱之後，您必須遵循相同步驟以掛接主要機箱。

> [!NOTE]
> * 主要機箱與 EBOD 機箱之間的機架中可能會有一些空的插槽。
> * 使用提供的 2m SAS 纜線以連接主要機箱與 EBOD 機箱。
> * EBOD 單元的前端單元相對位置沒有限制。 因此，主要機箱可以放在最上層插槽中，而 EBOD 機箱可以放在下方，或者反之亦然。
> 
> 

下一個步驟是針對裝置的電源、網路和序列存取進行佈線。

## <a name="cable-your-storsimple-8600-device"></a>佈線您的 StorSimple 8600 裝置
下列程序說明如何針對 StorSimple 8600 裝置的電源、網路和序列連線進行佈線。

### <a name="prerequisites"></a>先決條件
開始您的裝置佈線之前，您需要：

* 完全打開您的主要機箱與 EBOD 機箱的包裝
* 隨附於您的裝置的 4 條電源線 (主要及 EBOD 機箱各 2 條)
* 裝置隨附的 2 條 SAS 纜線以連接主要機箱與 EBOD 機箱
* 可以存取 2 個電源分配單元 (PDU) (建議)
* 網路纜線
* 提供的序列纜線
* 序列 USB 轉換器，且您的電腦上已安裝適當驅動程式 (如果需要)
* 提供 4 個 QSFP-to-SFP+ 配接器以用於 10 GbE 網路介面
* [10 GbE 網路介面在 StorSimple 裝置上支援的硬體](storsimple-supported-hardware-for-10-gbe-network-interfaces.md)

### <a name="sas-and-power-cabling"></a>SAS 及電源佈線
您的裝置同時有主要機箱和 EBOD 機箱。 這需要使用纜線將單元連接在一起，以取得序列連接 SCSI (SAS) 連線與電源。

第一次設定此裝置時，請先執行 SAS 佈線步驟，然後再完成電源佈線步驟。

[!INCLUDE [storsimple-cable-8600-for-SAS](../../includes/storsimple-sas-cable-8600.md)]

[!INCLUDE [storsimple-cable-8600-for-power](../../includes/storsimple-cable-8600-for-power.md)]

### <a name="network-cabling"></a>網路佈線
您的裝置是作用中待命組態：在任何指定時間，一個控制器模組為作用中，且在其他控制器模組待命時處理所有磁碟和網路作業。 如果控制器失敗，則待命控制器會立即啟動，並且繼續所有磁碟和網路作業。

若要支援此備援控制器容錯移轉，您需要如下列步驟所示佈線您的裝置網路。

#### <a name="to-cable-for-network-connection"></a>佈線網路連線
1. 您的裝置在每個控制器上有六個網路介面：四個 1 Gbps 和兩個 10 Gbps 乙太網路連接埠。 請參閱下圖以識別裝置後擋板上的各個資料連接埠。
   
     ![8600 裝置的後擋板](./media/storsimple-8600-hardware-installation/HCSBackplaneof2UDevicewithPortsLabeled.jpg)
   
    **裝置後方的資料連接埠**
   
   | 標籤 | 說明 |
   | --- | --- |
   |   0,1,4,5 |1 GbE 網路介面 |
   |   2,3 |10 GbE 網路介面 |
   |   6 |序列連接埠 |
2. 請參閱下圖的網路佈線。 (最小的網路組態會以藍色實線顯示。 對於高可用性和效能，需要的其他組態會以虛線顯示。)

![為您的 4U 裝置進行網路佈線](./media/storsimple-8600-hardware-installation/HCSCableYour4UDeviceforNetwork.png)

**您裝置的網路纜線**

| 標籤 | 說明 |
| --- | --- |
| 具有使用  |具有網際網路存取的 LAN |
| B |控制器 0 |
| C |PCM 0 |
| D |控制器 1 |
| E |PCM 1 |
| F |EBOD 控制器 0 |
| G |EBOD 控制器 1 |
| H,I |主機 (例如，檔案伺服器) |
| 0-5 |網路介面 |
| 6 |主要機箱 |
| 7 |EBOD 機箱 |

當連接裝置纜線時，最低設定需要：

* 各個控制器上至少已連接兩個網路介面，一個用於雲端存取，一個用於 iSCSI。 DATA 0 連接埠會自動啟用並透過裝置的序列主控台設定。 除了 DATA 0，另一個資料連接埠也需要透過 Azure 傳統入口網站來設定。 在此情況下，請將 DATA 0 連接埠連接到主要區域網路 (具有網際網路存取的網路)。 其他資料連接埠可以連線到網路的 SAN/iSCSI LAN (VLAN) 區段，視預期的角色而定。
* 各個控制器上的相同介面已連接到相同的網路以確保可用性 (如果發生控制器容錯移轉)。 例如，如果您選擇對於其中一個控制器連接 DATA 0 和 DATA 3，您需要在其他控制器上連接對應的 DATA 0 和 DATA 3。

針對高可用性和效能，請記住：

* 可能的話，請在各個控制器上設定一組用於雲端存取 (1 GbE)，和另一組用於 iSCSI (建議 10 GbE) 的網路介面。
* 可能的話，請將各個控制器的網路介面連接到兩個不同的交換器，以確保交換器發生錯誤時的可用性。 下圖說明兩個從各個控制器連接到兩個不同交換器的 10 GbE 網路介面 (DATA 2 和 DATA 3)。 如需詳細資訊，請參閱 **StorSimple 裝置的高可用性需求** 下的 [網路介面](storsimple-8000-system-requirements.md#high-availability-requirements-for-storsimple)。

> [!NOTE]
> 如果搭配使用 SFP + 收發器和您的 10 GbE 網路介面，請使用提供的 QSFP-SFP + 配接器。 如需詳細資訊，請移至 [10 GbE 網路介面在 StorSimple 裝置上支援的硬體](storsimple-supported-hardware-for-10-gbe-network-interfaces.md)。
> 
> 

### <a name="serial-port-cabling"></a>序列連接埠佈線
請執行下列步驟，以佈線您的序列連接埠。

#### <a name="to-cable-for-serial-connection"></a>佈線序列連線
1. 您的裝置在每個控制器上有以扳手圖示識別的序列連接埠。 若要找出序列連接埠的位置，請參閱顯示裝置背面資料連接埠的圖例。
2. 識別您的裝置後檔板上的作用中控制器。 閃爍的的藍色 LED 表示控制器作用中。
3. 使用提供的序列纜線 (如果需要，使用您的膝上型電腦的 USB-序列轉換器)，並將主控台或電腦 (具有裝置的終端機模擬) 連接到作用中控制器的序列連接埠。
4. 在電腦上安裝序列-USB 驅動程式 (隨附於裝置)。
5. 設定序列連線，如下所示：
   
   * 115,200 傳輸速率
   * 8 資料位元
   * 1 停止位元
   * 無同位檢查
   * 流量控制設為  **無**
6. 藉由在主控台上按下 Enter 鍵，驗證連線是否正在運作。 序列主控台功能表應該會出現。

> [!NOTE]
> **熄燈管理** ：當裝置安裝在遠端資料中心或在具有限制存取的電腦室時，請確定兩個控制器的序列連接一律會連線至序列主控台交換器或類似的設備。 如此可以在網路中斷或非預期失敗時允許頻外遠端控制和支援作業。
> 
> 

您已經完成您的裝置的電源、網路存取及序列連線的佈線。下一步是設定您的裝置上的軟體。

## <a name="next-steps"></a>後續步驟
您現在已準備好[部署和設定您的內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)。

