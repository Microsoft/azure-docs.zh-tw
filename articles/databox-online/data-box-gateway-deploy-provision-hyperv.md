---
title: 在 Hyper-V 中佈建 Azure 資料箱閘道的教學課程 | Microsoft Docs
description: 關於部署 Azure 資料箱閘道的第二個教學課程，牽涉到在 Hyper-V 中佈建虛擬裝置。
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 09/26/2018
ms.author: alkohli
ms.openlocfilehash: 3187c516ef22a97d2af89e08d5b99ad7a00d875d
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166452"
---
# <a name="tutorial-provision-azure-data-box-gateway-in-hyper-v-preview"></a>教學課程：在 Hyper-V 中佈建 Azure 資料箱閘道 (預覽)

## <a name="overview"></a>概觀

本教學課程說明如何在 Windows Server 2016、Windows Server 2012 R2 或 Windows Server 2012 上對執行 Hyper-V 的主機系統佈建資料箱閘道。 

您需要有系統管理員權限，才能佈建及設定虛擬裝置。 佈建及初始安裝程序可能需要大約 10 分鐘的時間才能完成。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 確定主機符合最低裝置需求
> * 在 Hypervisor 中佈建虛擬裝置
> * 啟動虛擬裝置，並取得 IP 位址

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

> [!IMPORTANT]
> - 資料箱閘道處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="prerequisites"></a>必要條件

在 Windows Server 2016 或 Windows Server 2012 R2 上對執行 Hyper-V 的主機系統佈建虛擬裝置的必要條件如下。

### <a name="for-the-data-box-gateway-resource"></a>針對資料箱閘道資源

在您開始前，請確定：

* 您已完成[準備入口網站以使用資料箱閘道](data-box-gateway-deploy-prep.md)中的所有步驟。
* 您已如[準備入口網站以使用資料箱閘道](data-box-gateway-deploy-prep.md)中所述，從 Azure 入口網站下載 Hyper-V 的虛擬裝置映像。

  > [!IMPORTANT]
  > 在資料箱閘道上執行的軟體只能用於資料箱閘道資源。
 
### <a name="for-the-data-box-gateway-virtual-device"></a>針對資料箱閘道虛擬裝置

在您部署裝置之前，請確定：

* 您可以存取可用來佈建裝置、在 Windows Server 2012 R2 或更新版本上執行 Hyper-V 的主機系統。
* 主機系統能夠把下列資源專門用來佈建虛擬裝置：

  * 至少 4 顆核心。
  * 至少 8 GB 的 RAM。 
  * 一個網路介面。
  * 250 GB 的 OS 磁碟。
  * 供資料使用的 2 TB 虛擬磁碟。

### <a name="for-the-network-in-the-datacenter"></a>針對資料中心內的網路

開始之前：

- 檢閱部署資料箱閘道的網路需求，並且根據每個需求設定資料中心網路。 如需詳細資訊，請參閱[資料箱閘道網路需求](data-box-gateway-system-requirements.md#networking-requirements)。
- 確定網際網路頻寬至少有 20 Mbps，以便讓裝置能夠發揮最大功能。


## <a name="check-the-host-system"></a>檢查主機系統

若要建立虛擬裝置，您需要：

* 安裝在 Windows Server 2016、Windows Server 2012 R2 或 Windows Server 2012 上的 Hyper-V 角色。
* 在已連線至主機的 Microsoft Windows 用戶端上的 Microsoft Hyper-V 管理員。
* 請確定要用來建立虛擬裝置的基本硬體 (主機系統)，能夠把下列資源專門提供給虛擬裝置來使用：

    * 至少 4 顆核心。
    * 至少 8 GB 的 RAM。
    * 網路介面，且已連線到能夠將流量路由至網際網路的網路。 
    * 250 GB 的 OS 磁碟。
    * 供系統資料使用的 2 TB 虛擬磁碟。

## <a name="provision-a-virtual-device-in-hypervisor"></a>在 Hypervisor 中佈建虛擬裝置

請執行下列步驟，以便在您的 Hypervisor 中佈建裝置。

1. 在 Windows Server 主機上，將虛擬裝置映像複製到本機磁碟機中。 您已透過 Azure 入口網站下載這個 VHDX 映像。 請記下您複製映像的位置，因為稍後會在程序中使用此映像。
2. 開啟 [伺服器管理員] 。 按一下右上角的 [工具]，然後選取 [Hyper-V 管理員]。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image1.png)  
  
3. 在 [Hyper-V 管理員] 的範圍窗格中，於您的系統節點上按一下滑鼠右鍵以開啟操作功能表，然後按一下 [新增]  >  [虛擬機器]。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image2.png)
4. 在「新增虛擬機器精靈」的「開始之前」頁面上，按 [下一步]。
5. 在「指定名稱和位置」頁面上，提供您虛擬裝置的「名稱」。 按 [下一步] 。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image3.png)
6. 在 [指定世代] 頁面上，選擇 [第二代] 作為 .vhdx 裝置映像類型，然後按 [下一步]。    

   ![](./media/data-box-gateway-deploy-provision-hyperv/image4.png)
7. 在 [指派記憶體] 頁面上，至少指定 **8192 MB** 的「啟動記憶體」，請勿啟用動態記憶體，然後按一下 [下一步]。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image5.png) 
8. 在 [設定網路功能] 頁面上，指定連線到網際網路的虛擬交換器，然後按一下 [下一步]。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image6.png)
9. 在「連接虛擬硬碟」 頁面上，選擇 [使用現有的虛擬硬碟]，指定虛擬裝置映像的位置，然後按 [下一步]。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image7.png)
10. 檢閱「摘要」，然後按一下 [結束] 來建立虛擬機器。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image8.png)
11. 您需要 4 顆核心才能符合最低需求。 若要新增 4 顆虛擬處理器，請在 [Hyper-V 管理員] 視窗中選取您的主機系統。 在右窗格中的 [虛擬機器] 清單下，找出您剛才建立的虛擬機器。 選取虛擬機器的名稱並按一下滑鼠右鍵，然後選取 [設定] 。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image9.png)
12. 在 [設定] 頁面上，按一下左窗格中的 [處理器]。 在右窗格中，將 [虛擬處理器數目] 設定為 4 (或更多)。 按一下 [套用]。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image10.png)
13. 您也必須新增 2 TB 的虛擬資料磁碟，才能符合最低需求。 在 [設定] 頁面中：

    1. 在左窗格中，選取 [SCSI 控制器]。
    2. 在右窗格中，選取 [硬碟]，然後按一下 [新增]。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image11.png)
14. 在 [硬碟] 頁面上，選取 [虛擬硬碟] 選項，然後按一下 [新增]。 [新增虛擬硬碟精靈] 會啟動。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image12.png)
1. 在「新增虛擬硬碟精靈」的 [開始之前] 頁面上，按一下 [下一步]。
2. 在 [選擇磁碟格式] 頁面上，接受 [VHDX] 格式預設選項。 按 [下一步] 。
   
17. 在 [選擇磁碟類型] 頁面上，將虛擬硬碟類型設定為 [動態擴充] \(建議選項)。 [固定大小] 磁碟可以運作，但您可能需要等待很長一段時間。 建議您不要使用 [差異]  選項。 按 [下一步] 。 

    ![](./media/data-box-gateway-deploy-provision-hyperv/image13.png)
18. 在 [指定名稱和位置] 頁面上，提供資料磁碟的「名稱」和「位置」(您可以瀏覽至該位置)。 按 [下一步] 。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image14.png)
19. 在「設定磁碟」頁面上，選取 [建立新的空白虛擬硬碟] 選項，然後將大小指定為 **2 TB** (或更多)。 2 TB 是最低需求，您永遠可以佈建更大的磁碟。 請注意，佈建之後您無法縮小磁碟。  但您可以藉由新增資料磁碟來擴充磁碟。 按 [下一步] 。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image15.png)
20. 在 [摘要] 頁面上，檢閱虛擬資料磁碟的詳細資料，如果您對這些資料感到滿意，請按一下 [完成] 來建立磁碟。 精靈會關閉，虛擬硬碟會新增至您的電腦。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image16.png)
21. 返回 [設定] 頁面。 按一下 [確定] 來關閉「設定」頁面並返回 [Hyper-V 管理員] 視窗。

    ![](./media/data-box-gateway-deploy-provision-hyperv/image17.png)

## <a name="start-the-virtual-device-and-get-the-ip"></a>啟動虛擬裝置並取得 IP 位址
請執行下列步驟來啟動您的虛擬裝置，並連線到該虛擬裝置。

#### <a name="to-start-the-virtual-device"></a>啟動虛擬裝置
1. 啟動虛擬裝置。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image18.png)
2. 當裝置開始執行之後，選取裝置並按一下滑鼠右鍵，然後選取 [連接] 。

3. 您可能需要等待 10 至 15 分鐘，裝置才會準備就緒。 主控台會顯示狀態訊息以表明進度。 裝置就緒之後，請前往 [動作] 。 按 `Ctrl + Alt + Delete` 來登入虛擬裝置。 預設使用者為 EdgeUser，預設密碼為 Password1。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image21.png)
   
6. 步驟 5 至 7 僅適用於在非 DHCP 環境中開機的情況。 如果您是在 DHCP 環境中，請略過這些步驟。 如果您是在非 DHCP 環境中讓裝置開機，您會看到此效果的訊息。
    
7. 若要設定網路，請使用 `Get-HcsIpAddress` 命令來列出虛擬裝置上已啟用的網路介面。 如果您的裝置有已啟用的單一網路介面，系統指派給該介面的預設名稱會是 `Ethernet`。

8. 使用 `Set-HcsIpAddress` Cmdlet 來設定網路。 請參閱下列範例：

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`
    
9. 初始安裝程序完成，且裝置已開機之後，您將會看到裝置橫幅文字。 請記下 IP 位址及橫幅文字中的 URL，以便管理裝置。 使用此 IP 位址連線到虛擬裝置的 Web UI，以及完成本機安裝和啟用。

   ![](./media/data-box-gateway-deploy-provision-hyperv/image23.png)
      

如果裝置不符合最低設定需求，橫幅文字中會出現錯誤訊息。 請修改裝置設定，讓電腦有足夠的資源符合最低需求。 然後您就可以將裝置重新啟動，並連線到該裝置。 請參閱[確認主機系統符合最低虛擬裝置需求](#check-the-host-system)中的最低組態需求。

<!--If you face any other error during the initial configuration using the local web UI, refer to the following workflows:

* Run diagnostic tests to [troubleshoot web UI setup](storsimple-ova-web-ui-admin.md#troubleshoot-web-ui-setup-errors).
* [Generate log package and view log files](storsimple-ova-web-ui-admin.md#generate-a-log-package).-->

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解資料箱閘道的相關主題，像是：

> [!div class="checklist"]
> * 確定主機符合最低裝置需求
> * 在 Hypervisor 中佈建虛擬裝置
> * 啟動虛擬裝置，並取得 IP 位址

前進至下一個教學課程，以了解如何連線、設定及啟動您的虛擬裝置。

> [!div class="nextstepaction"]
> [連線並設定您的資料箱閘道](./data-box-gateway-deploy-connect-setup-activate.md)


