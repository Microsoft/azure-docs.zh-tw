---
title: 如何在 SAP HANA on Azure (大型執行個體) 上安裝 HANA | Microsoft Docs
description: 如何在 SAP HANA on Azure (大型執行個體) 上安裝 HANA。
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 76a7ce99799b85d81aa6e127ebe1e57e2df3e59a
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44167588"
---
# <a name="example-of-an-sap-hana-installation-on-hana-large-instances"></a>範例：在 HANA 大型執行個體上安裝 SAP HANA

本節示範如何在 HANA 大型執行個體單位上安裝 SAP HANA。 開始的狀態看起來像：

- 您向 Microsoft 提供所有資料以部署 SAP HANA 大型執行個體。
- 您從 Microsoft 收到 SAP HANA 大型執行個體。
- 您建立了連線到內部部署網路的 Azure VNet。
- 您將 HANA 大型執行個體的 ExpressRotue 線路連線到相同的 Azure VNet。
- 您安裝了 Azure VM 用做為 HANA 大型執行個體的跳躍方塊。
- 您很確定可以從跳躍方塊連線到 HANA 大型執行個體單位，反之亦然。
- 您已確認是否已安裝所有必要的套件及修補程式。
- 您閱讀了在所用作業系統上安裝 HANA 的相關 SAP 附註和文件，並確定作業系統版本支援 HANA 版本選擇。

接下來顯示的內容是將 HANA 安裝套件下載到跳躍方塊 VM，本例是在 Windows 作業系統中執行，HANA 大型執行個體單位的套件複本和安裝順序。

## <a name="download-of-the-sap-hana-installation-bits"></a>下載 SAP HANA 安裝位元組
因為 HANA 大型執行個體單位不直接連線到網際網路，所以您無法直接從 SAP 將安裝套件下載到 HANA 大型執行個體 VM。 為解決沒有直接網際網路連線的問題，您需要跳躍方塊。 您將套件下載到跳躍方塊 VM。

為下載 HANA 安裝套件，您需要 SAP S-user 或其他使用者，讓您存取 SAP Marketplace。 登入之後，請完成這一系列的畫面：

移至 [SAP Service Marketplace](https://support.sap.com/en/index.html) > 按一下 [下載軟體] > [Installations and Upgrade] \(安裝與升級) > [By Alphabetical Index] \(依字母順序排列索引) > 在 H – SAP HANA 平台版本下 > SAP HANA 平台版本 2.0 > [安裝] > 下載下列檔案

![下載 HANA 安裝](./media/hana-installation/image16_download_hana.PNG)

在示範案例中，我們下載了 SAP HANA 2.0 安裝套件。 在 Azure 跳躍方塊 VM 中，將自我解壓縮的封存展開到目錄中，如下所示。

![解壓縮 HANA 安裝](./media/hana-installation/image17_extract_hana.PNG)

解壓縮封存後，將 HANA 大型執行個體單位的解壓縮目錄 (上例中為 51052030)，複製到您所建目錄的 /hana/shared 磁碟區。

> [!Important]
> 安裝套件「請勿」複製到根或開機 LUN，因為空間有限，而且必須供其他處理序使用。


## <a name="install-sap-hana-on-the-hana-large-instance-unit"></a>在 HANA 大型執行個體單位上安裝 SAP HANA
為安裝 SAP HANA，您需要以根使用者身分登入。 只有根才有足夠的權限安裝 SAP HANA。
您需要做的第一件事就是設定複製的 /hana/shared 目錄權限。 需要設定的權限類似

```
chmod –R 744 <Installation bits folder>
```

如果您想要使用圖形化設定安裝 SAP HANA，需要將 gtk2 套件安裝在 HANA 大型執行個體上。 檢查是否已使用命令安裝

```
rpm –qa | grep gtk2
```

在後續步驟中，我們會示範使用圖形化使用者介面安裝 SAP HANA 的過程。 在下個步驟中，移至安裝目錄並瀏覽至子目錄 HDB_LCM_LINUX_X86_64。 Start

```
./hdblcmgui 
```
從該目錄。 現在您要跟著指引完成一連串的畫面，並提供安裝資料。 在所示範例中，我們要安裝 SAP HANA 資料庫伺服器和 SAP HANA 用戶端元件。 因此我們選取 [SAP HANA 資料庫]，如下所示。

![在安裝中選取 [HANA]](./media/hana-installation/image18_hana_selection.PNG)

在下一個畫面中選擇 [Install New System] \(安裝新系統) 選項

![新安裝選取 [HANA]](./media/hana-installation/image19_select_new.PNG)

這個步驟之後，您需要在數個不同的其他元件中選取，它們都可以另外安裝至 SAP HANA 資料庫伺服器。

![選取其他的 HANA 元件](./media/hana-installation/image20_select_components.PNG)

為達成本文件的目標，我們選擇 SAP HANA 用戶端和 SAP HANA Studio。 我們也安裝了相應增加執行個體。 因此在下一個畫面中，您需要選擇 [Single-Host System] \(單一主機系統)。 

![選取相應增加安裝](./media/hana-installation/image21_single_host.PNG)

在下一個畫面中，您需要提供一些資料。

![提供 SAP HANA SID](./media/hana-installation/image22_provide_sid.PNG)

> [!Important]
> 您需要提供在訂購 HANA 大型執行個體部署時提供給 Microsoft 的相同 SID，當作 HANA 系統識別碼 (SID)。 選擇不同的 SID，安裝會因為不同磁碟區上的存取權限問題而失敗。

使用 /hana/shared 目錄當做安裝目錄。 在下一個步驟中，您需要提供 HANA 資料檔和 HANA 記錄檔的位置。


![提供 HANA 記錄檔位置](./media/hana-installation/image23_provide_log.PNG)

> [!Note]
> 您應該將隨附於掛接點、包含前一個畫面所選 SID 的磁碟區，定義為資料和記錄檔磁碟區。 如果 SID 與您鍵入的不相符，請在前一個畫面中，返回並調整 SID，使與掛接點上的值相合。

在下一個步驟中，檢閱主機名稱並加以更正。 

![檢閱主機名稱](./media/hana-installation/image24_review_host_name.PNG)

在下一個步驟中，您也需要擷取在訂購 HANA 大型執行個體部署時提供給 Microsoft 的資料。 

![提供系統使用者 UID 與 GID](./media/hana-installation/image25_provide_guid.PNG)

> [!Important]
> 您需要提供在訂購單位部署時提供給 Microsoft 的相同系統使用者識別碼和使用者群組識別碼。 如果無法提供完全一致的識別碼，在 HANA 大型執行個體單位上安裝 SAP HANA 就會失敗。

在接下來的兩個畫面中 (本文不顯示)，您需要提供 SAP HANA 資料庫的系統使用者密碼和 sapadm 使用者密碼，以將 SAP 主機代理程式安裝為 SAP HANA 資料庫執行個體的一部分。

定義密碼之後，會出現確認畫面。 請檢查所有列出的資料列，再繼續進行安裝。 您到達記錄安裝進度的進度畫面，類似下面這個：

![檢查安裝進度](./media/hana-installation/image27_show_progress.PNG)

當安裝完成後，您應該會看到類似下圖的畫面。

![安裝完成](./media/hana-installation/image28_install_finished.PNG)

此時，SAP HANA 執行個體應該啟動並執行，隨時可供使用。 您應該能夠從 SAP HANA Studio 連接到它。 也請務必要檢查並套用最新的 SAP HANA 修補程式。


**後續步驟**

- 請參閱 [Azure 上的 SAP HANA 大型執行個體高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md)。

