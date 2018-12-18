---
title: 在 Hyper-V 中佈建 StorSimple Virtual Array | Microsoft Docs
description: 這是 StorSimple Virtual Array 部署中的第二個教學課程，說明在 Hyper-V 中佈建虛擬陣列。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 4354963c-e09d-41ac-9c8b-f21abeae9913
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/15/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: bad431c8958f7d381bb9c0410caa3a57c6e75c19
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23111076"
---
# <a name="deploy-storsimple-virtual-array---provision-in-hyper-v"></a>部署 StorSimple Virtual Array：在 Hyper-V 中佈建
![](./media/storsimple-virtual-array-deploy2-provision-hyperv/hyperv4.png)

## <a name="overview"></a>概觀
本教學課程說明如何在執行 Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 之 Hyper-V 的主機系統上佈建 StorSimple Virtual Array。 本文適用於在 Azure 入口網站及 Microsoft Azure 政府服務雲端部署 StorSimple Virtual Array。

您需要有系統管理員權限，才能佈建及設定虛擬陣列。 佈建及初始安裝程序可能需要大約 10 分鐘的時間才能完成。

## <a name="provisioning-prerequisites"></a>佈建的必要條件
此處提供在執行 Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 之 Hyper-V 的主機系統上佈建虛擬陣列的必要條件。

### <a name="for-the-storsimple-device-manager-service"></a>StorSimple 裝置管理員服務
在您開始前，請確定：

* 您已完成 [準備入口網站以使用 StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md)中的所有步驟。
* 您已經從 Azure 入口網站下載適用於 Hyper-V 的虛擬陣列映像。 如需詳細資訊，請參閱[準備入口網站以使用 StorSimple Virtual Array 指南](storsimple-virtual-array-deploy1-portal-prep.md)的**步驟 3︰下載虛擬陣列映像**。

  > [!IMPORTANT]
  > StorSimple Virtual Array 上執行的軟體只能搭配 Storsimple 裝置管理員服務來使用。
  >
  >

### <a name="for-the-storsimple-virtual-array"></a>StorSimple Virtual Array
在您部署虛擬陣列之前，請確定：

* 您可以存取可用來佈建裝置、在 Windows Server 2008 R2 或更新版本上執行 Hyper-V 的主機系統。
* 主機系統能夠把下列資源專門用來佈建虛擬陣列：

  * 至少 4 顆核心。
  * 至少 8 GB 的 RAM。 若計畫將虛擬陣列設定為檔案伺服器，8 GB 支援 2 百萬個以下的檔案。 您需要 16 GB RAM 才能支援 2 - 4 百萬個檔案。
  * 一個網路介面。
  * 供資料使用的 500 GB 虛擬磁碟。

### <a name="for-the-network-in-the-datacenter"></a>針對資料中心內的網路
開始之前，請檢閱部署 StorSimple Virtual Array 的網路需求，並適當地設定資料中心網路。 如需詳細資訊，請參閱 [StorSimple Virtual Array 網路需求](storsimple-ova-system-requirements.md#networking-requirements)。

## <a name="step-by-step-provisioning"></a>佈建的逐步指示
若要佈建並連接至虛擬陣列，您必須執行下列步驟：

1. 確定主機系統有足夠的資源符合最低的虛擬陣列需求。
2. 在 Hypervisor 中佈建虛擬陣列。
3. 啟動虛擬陣列並取得 IP 位址。

我們會在下列各節解釋這些步驟。

## <a name="step-1-ensure-that-the-host-system-meets-minimum-virtual-array-requirements"></a>步驟 1：確定主機系統符合最低的虛擬陣列需求
若要建立虛擬陣列，您需要︰

* 安裝在 Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 SP1 上的 Hyper-V 角色。
* 在已連線至主機的 Microsoft Windows 用戶端上的 Microsoft Hyper-V 管理員。

請確定您要建立虛擬陣列的基礎硬體 (主機系統) 能夠把下列資源專門提供給虛擬陣列來使用：

* 至少 4 顆核心。
* 至少 8 GB 的 RAM。 若計畫將虛擬陣列設定為檔案伺服器，8 GB 支援 2 百萬個以下的檔案。 您需要 16 GB RAM 才能支援 2 - 4 百萬個檔案。
* 一個網路介面。
* 供系統資料使用的 500 GB 虛擬磁碟。

## <a name="step-2-provision-a-virtual-array-in-hypervisor"></a>步驟 2：在 Hypervisor 中佈建虛擬陣列
請執行下列步驟，以便在您的 Hypervisor 中佈建裝置。

#### <a name="to-provision-a-virtual-array"></a>若要佈建虛擬陣列
1. 在 Windows Server 主機上，將虛擬陣列映像複製到本機磁碟機。 您已透過 Azure 入口網站下載此映像 (VHD 或 VHDX)。 請記下您複製映像的位置，因為稍後會在程序中使用此映像。
2. 開啟 [伺服器管理員] 。 按一下右上角的 [工具]，然後選取 [Hyper-V 管理員]。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image1.png)  

   如果您執行的是 Windows Server 2008 R2，請開啟「Hyper-V 管理員」。 在 [伺服器管理員] 中，按一下 [角色] > [Hyper-V] > [Hyper-V 管理員]。
3. 在 [Hyper-V 管理員] 的範圍窗格中，於您的系統節點上按一下滑鼠右鍵以開啟操作功能表，然後按一下 [新增]  >  [虛擬機器]。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image2.png)
4. 在「新增虛擬機器精靈」的 [開始之前] 頁面上，按一下 [下一步]。
5. 在 [指定名稱和位置] 頁面上，提供虛擬陣列的 [名稱]。 按 [下一步] 。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image4.png)
6. 在 [指定世代] 頁面上，選擇裝置映像類型，然後按一下 [下一步]。 如果您使用的是 Windows Server 2008 R2，則不會出現此頁面。

   * 如果您已下載適用於 Windows Server 2012 或更新版本的 .vhdx 映像，請選擇 [第 2 代]  。
   * 如果您已下載適用於 Windows Server 2008 R2 或更新版本的 .vhd 映像，請選擇 [第 1 代]  。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image5.png)
7. 在 [指派記憶體] 頁面上，至少指定 **8192 MB** 的「啟動記憶體」，請勿啟用動態記憶體，然後按一下 [下一步]。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image6.png)  
8. 在 [設定網路功能] 頁面上，指定連線到網際網路的虛擬交換器，然後按一下 [下一步]。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image7.png)
9. 在 [連接虛擬硬碟] 頁面上，選擇 [使用現有的虛擬硬碟]，指定虛擬陣列映像 (.vhdx 或.vhd) 的位置，然後按一下 [下一步]。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image8m.png)
10. 檢閱「摘要」，然後按一下 [結束] 來建立虛擬機器。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image9.png)
11. 您需要 4 顆核心才能符合最低需求。 若要新增 4 顆虛擬處理器，請在 [Hyper-V 管理員] 視窗中選取您的主機系統。 在右窗格中的 [虛擬機器] 清單下，找出您剛才建立的虛擬機器。 選取虛擬機器的名稱並按一下滑鼠右鍵，然後選取 [設定] 。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image10.png)
12. 在 [設定] 頁面上，按一下左窗格中的 [處理器]。 在右窗格中，將 [虛擬處理器數目] 設定為 4 (或更多)。 按一下 [套用]。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image11.png)
13. 您也必須新增 500 GB 的虛擬資料磁碟，才能符合最低需求。 在 [設定] 頁面中：

    1. 在左窗格中，選取 [SCSI 控制器]。
    2. 在右窗格中，選取 [硬碟]，然後按一下 [新增]。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image12.png)
14. 在 [硬碟] 頁面上，選取 [虛擬硬碟] 選項，然後按一下 [新增]。 [新增虛擬硬碟精靈] 會啟動。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image13.png)
15. 在「新增虛擬硬碟精靈」的 [開始之前] 頁面上，按一下 [下一步]。
16. 在 [選擇磁碟格式] 頁面上，接受 [VHDX] 格式預設選項。 按 [下一步] 。 如果您執行的是 Windows Server 2008 R2，則不會出現此畫面。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image15.png)
17. 在 [選擇磁碟類型] 頁面上，將虛擬硬碟類型設定為 [動態擴充] \(建議選項)。 [固定大小] 磁碟可以運作，但您可能需要等待很長一段時間。 建議您不要使用 [差異]  選項。 按 [下一步] 。 在 Windows Server 2012 R2 和 Windows Server 2012 中，[動態擴充] 是預設選項，而在 Windows Server 2008 R2 中，預設值是 [固定大小]。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image16.png)
18. 在 [指定名稱和位置] 頁面上，提供資料磁碟的「名稱」和「位置」(您可以瀏覽至該位置)。 按 [下一步] 。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image17.png)
19. 在「設定磁碟」頁面上，選取 [建立新的空白虛擬硬碟] 選項，然後將大小指定為 **500 GB** (或更多)。 500 GB 是最低需求，您永遠可以佈建更大的磁碟。 請注意，佈建之後您無法擴充或縮小磁碟。 如需有關要佈建之磁碟大小的詳細資訊，請檢閱[最佳作法文件](storsimple-ova-best-practices.md)中的＜調整大小＞一節。 按 [下一步] 。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image18.png)
20. 在 [摘要] 頁面上，檢閱虛擬資料磁碟的詳細資料，如果您對這些資料感到滿意，請按一下 [完成] 來建立磁碟。 精靈會關閉，虛擬硬碟會新增至您的電腦。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image19.png)
21. 返回 [設定] 頁面。 按一下 [確定] 來關閉「設定」頁面並返回 [Hyper-V 管理員] 視窗。

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image20.png)

## <a name="step-3-start-the-virtual-array-and-get-the-ip"></a>步驟 3：啟動虛擬陣列並取得 IP
請執行下列步驟來啟動並連接至虛擬陣列。

#### <a name="to-start-the-virtual-array"></a>若要啟動虛擬陣列
1. 啟動虛擬陣列。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image21.png)
2. 當裝置開始執行之後，選取裝置並按一下滑鼠右鍵，然後選取 [連接] 。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image22.png)
3. 您可能需要等待 5 至 10 分鐘，裝置才會準備就緒。 主控台會顯示狀態訊息以表明進度。 裝置就緒之後，請前往 [動作] 。 按 `Ctrl + Alt + Delete` 來登入虛擬陣列。 預設使用者為 *StorSimpleAdmin*，預設密碼為 *Password1*。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image23.png)
4. 基於安全性理由，裝置系統管理員密碼會在第一次登入時過期。 系統會提示您變更密碼。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image24.png)

   請輸入至少包含 8 個字元的密碼。 密碼必須至少符合下列 4 個需求中的 3 個：大寫、小寫、數字和特殊字元。 請重新輸入密碼來加以確認。 系統會通知您密碼已經變更。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image25.png)
5. 成功變更密碼之後，虛擬陣列可能會重新啟動。 請等待裝置啟動。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image26.png)

    畫面會出現裝置的 Windows PowerShell 主控台及進度列。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image27.png)
6. 步驟 6 至 8 僅適用於在非 DHCP 環境中開機的情況。 如果您是在 DHCP 環境中，請略過這些步驟並前往步驟 9。 如果您是在非 DHCP 環境中讓裝置開機，您會看到下列畫面。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image28m.png)

    接著，設定網路。
7. 使用 `Get-HcsIpAddress` 命令來列出虛擬陣列上已啟用的網路介面。 如果您的裝置有已啟用的單一網路介面，系統指派給該介面的預設名稱會是 `Ethernet`。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image29m.png)
8. 使用 `Set-HcsIpAddress` Cmdlet 來設定網路。 請參閱下列範例：

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

    ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image30.png)
9. 初始安裝程序完成，且裝置已開機之後，您將會看到裝置橫幅文字。 請記下 IP 位址及橫幅文字中的 URL，以便管理裝置。 使用此 IP 位址連接至虛擬陣列的 Web UI，並完成本機設定和註冊。

   ![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image31m.png)
10. (選擇性) 只有當您要在政府服務雲端部署裝置時，才執行這個步驟。 您現在將在您的裝置上啟用美國聯邦資訊處理標準 (FIPS) 模式。 FIPS 140 標準定義核准美國聯邦政府電腦系統所使用的密碼編譯演算法來保護機密資料。

    1. 若要啟用 FIPS 模式，請執行下列 Cmdlet：

        `Enable-HcsFIPSMode`
    2. 在您啟用 FIPS 模式之後，重新啟動您的裝置，讓密碼編譯驗證生效。

       > [!NOTE]
       > 您可以在您的裝置上啟用或停用 FIPS 模式。 不支援在 FIPS 和非 FIPS 模式之間替換裝置。
       >
       >

如果裝置不符合最低設定需求，橫幅文字中會出現下列錯誤 (如下所示)。 請修改裝置設定，讓電腦有足夠的資源符合最低需求。 然後您就可以將裝置重新啟動，並連線到該裝置。 請參閱[步驟 1：確定主機系統符合最低的虛擬陣列需求](#step-1-ensure-that-the-host-system-meets-minimum-virtual-device-requirements)中的最低設定需求。

![](./media/storsimple-virtual-array-deploy2-provision-hyperv/image32.png)

如果您在使用本機 Web UI 進行初始設定時碰到其他任何錯誤，請參閱下列工作流程：

* 執行診斷測試來 [疑難排解 Web UI 安裝程式錯誤](storsimple-ova-web-ui-admin.md#troubleshoot-web-ui-setup-errors)。
* [產生記錄檔封裝及檢視記錄檔](storsimple-ova-web-ui-admin.md#generate-a-log-package)。

## <a name="next-steps"></a>後續步驟
* [Set up your StorSimple Virtual Array as a file server (將 StorSimple 虛擬陣列設定為檔案伺服器)](storsimple-virtual-array-deploy3-fs-setup.md)
* [Set up your StorSimple Virtual Array as an iSCSI server (將 StorSimple 虛擬陣列設定為 iSCSI 伺服器)](storsimple-virtual-array-deploy3-iscsi-setup.md)
