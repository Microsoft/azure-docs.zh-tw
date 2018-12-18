---
title: 在 VMware 中佈建 Azure 資料箱閘道的教學課程 | Microsoft Docs
description: 關於部署 Azure 資料箱閘道的第二個教學課程，牽涉到在 VMware 中佈建虛擬裝置。
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 10/01/2018
ms.author: alkohli
ms.openlocfilehash: ea4203c45f482b990122a966fc2ec13b3fb41c84
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167149"
---
# <a name="tutorial-provision-azure-data-box-gateway-in-vmware-preview"></a>教學課程：在 VMware 中佈建 Azure 資料箱閘道 (預覽)

## <a name="overview"></a>概觀

本教學課程說明如何在執行 VMware ESXi 6.0 或 6.5 的主機系統上佈建資料箱閘道。 

您需要有系統管理員權限，才能佈建並連接至虛擬裝置。 佈建及初始安裝程序可能需要大約 10 分鐘的時間才能完成。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 確定主機符合最低裝置需求
> * 在 VMware 中佈建虛擬裝置
> * 啟動虛擬裝置，並取得 IP 位址

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

> [!IMPORTANT]
> - 資料箱閘道處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="prerequisites"></a>必要條件

在執行 VMware ESXi 6.0 或 6.5 的主機系統上，佈建虛擬裝置的必要條件如下。

### <a name="for-the-data-box-gateway-resource"></a>針對資料箱閘道資源

在您開始前，請確定：

* 您已完成[準備入口網站以使用資料箱閘道](data-box-gateway-deploy-prep.md)中的所有步驟。
* 您已如[準備入口網站以使用資料箱閘道](data-box-gateway-deploy-prep.md)中所述，從 Azure 入口網站下載 VMware 的虛擬裝置映像。

  > [!IMPORTANT]
  > 在資料箱閘道上執行的軟體只能用於資料箱閘道資源。

### <a name="for-the-data-box-gateway-virtual-device"></a>針對資料箱閘道虛擬裝置

在您部署虛擬裝置之前，請確定：

* 您可以存取執行 VMware (ESXi 6.0 或 6.5) 且可用來佈建裝置的主機系統。
* 主機系統能夠把下列資源專門用來佈建虛擬裝置：

  * 至少 4 顆核心。
  * 至少 8 GB 的 RAM。
  * 一個網路介面。
  * 250 GB 的 OS 磁碟。
  * 供系統資料使用的 2 TB 虛擬磁碟。

### <a name="for-the-network-in-datacenter"></a>對於資料中心的網路

開始之前：

- 檢閱部署資料箱閘道的網路需求，並且根據每個需求設定資料中心網路。 如需詳細資訊，請參閱[資料箱閘道網路需求](data-box-gateway-system-requirements.md#networking-requirements)。
- 確定網際網路頻寬至少有 20 Mbps，以便讓裝置能夠發揮最大功能。

## <a name="check-the-host-system"></a>檢查主機系統

若要建立虛擬裝置，您需要：

* 能夠存取執行 VMware ESXi 伺服器 6.0 或 6.5 的主機系統。 主機系統能夠把下列資源專門用於虛擬裝置：
 
  * 至少 4 顆核心。
  * 至少 8 GB 的 RAM。 
  * 網路介面，且已連線到能夠將流量路由至網際網路的網路。 
  * 250 GB 的 OS 磁碟。
  * 供資料使用的 2 TB 虛擬磁碟。
* 您系統上的 VMware vSphere 用戶端，以便管理 ESXi 主機。


## <a name="provision-a-virtual-device-in-hypervisor"></a>在 Hypervisor 中佈建虛擬裝置

請執行下列步驟，以便在您的 Hypervisor 中佈建虛擬裝置。

1. 複製您系統中的虛擬裝置映像。 您已透過 Azure 入口網站下載這個虛擬映像 (兩個檔案)。 請記下您複製映像的位置，因為稍後會在程序中使用此映像。

2. 使用 vSphere Web 用戶端登入 ESXi 伺服器。 您需要有系統管理員權限，才能建立虛擬機器。

   ![](./media/data-box-gateway-deploy-provision-vmware/image1.png)
  
3. 將 VMDK 上傳至 ESXi 伺服器。 在 [導覽] 窗格中，選取 [儲存體]。

   ![](./media/data-box-gateway-deploy-provision-vmware/image2.png)

4. 在右窗格的 [資料存放區] 下方，選取您要上傳 VMDK 的資料存放區。 

    - 資料存放區的類型必須是 VMFS5。 
    - 資料存放區也必須要有足夠的可用空間來容納作業系統和資料磁碟。
   
5. 按一下滑鼠右鍵並選取 [瀏覽資料存放區]。

   ![](./media/data-box-gateway-deploy-provision-vmware/image3.png)

6. 畫面會出現 [資料存放區瀏覽器]  視窗。

   ![](./media/data-box-gateway-deploy-provision-vmware/image4.png)

7. 在工具列上，按一下 [建立目錄] 圖示來建立新資料夾。 然後指定資料夾的名稱，並把該名稱記下來。 您稍後在建立虛擬機器時，將會使用該資料夾名稱 (建議的最佳做法)。 按一下 [建立目錄]。

   ![](./media/data-box-gateway-deploy-provision-vmware/image5.png)

8. [資料存放區瀏覽器] 的左窗格中會出現新的資料夾。 按一下 [上傳] 圖示，然後選取 [上傳檔案]。

    ![](./media/data-box-gateway-deploy-provision-vmware/image6.png)

9. 瀏覽並指向您已下載的 VMDK 檔案。 有兩個檔案。 選取要上傳的檔案。

    ![](./media/data-box-gateway-deploy-provision-vmware/image7.png)

10. 按一下 [開啟] 。 就會開始將 VMDK 檔案上傳至指定的資料存放區。 檔案可能需要幾分鐘的時間才能上傳完畢。
11. 上傳完成之後，檔案就會出現在資料存放區裡您所建立的資料夾中。 現在將第二個 VMDK 檔案上傳至相同的資料存放區。 一旦兩個檔案都上傳，這兩個檔案就會合併成單一檔案。 然後，您會在目錄中看到單一檔案。

    ![](./media/data-box-gateway-deploy-provision-vmware/image8.png)

12. 返回 vSphere 用戶端視窗。 在 [導覽] 窗格中，選取 [虛擬機器]。 在右窗格中，按一下 [建立/註冊 VM]。

    ![](./media/data-box-gateway-deploy-provision-vmware/image9.png)

13. [新增虛擬機器] 隨即顯示。 在 [選取] 建立類型底下，選擇 [建立新的虛擬機器]，然後按 [下一步]。
    ![](./media/data-box-gateway-deploy-provision-vmware/image10.png)

14. 在 [選取名稱和 OS 名稱及位置] 頁面上，指定虛擬機器的**名稱**。 這個名稱應該與之前在步驟 7 中指定的資料夾名稱 (建議的最佳做法) 相同。 選擇 Windows 作為 [客體 OS 系列]，選擇 Microsoft Windows Server 2016 (64 位元) 作為 [客體 OS 版本]。 按 [下一步] 。

    ![](./media/data-box-gateway-deploy-provision-vmware/image11.png)

15. 在 [選取儲存體]  頁面上，選取您要用來佈建虛擬機器的資料存放區。 按 [下一步] 。

    ![](./media/data-box-gateway-deploy-provision-vmware/image12.png)
16. 在 [自訂設定] 頁面上，將 [CPU] 設為 4、將 [記憶體] 設為 8192 MB (或以上)、將 [硬碟 1] 設為 2 TB (或以上)。 選擇要新增的 **SCSI 硬碟**。 在此案例中，是 LSI Logic SAS。 **不支援靜態 IDE 磁碟。** [硬碟 1] 是虛擬資料磁碟。 請注意，佈建之後您無法縮小磁碟。

    ![](./media/data-box-gateway-deploy-provision-vmware/image13.png)

    在相同頁面上，按一下 [新增硬碟]，然後選取 [現有硬碟]。 選取資料存放區中的 VMDK 檔案。 這樣會新增 OS 磁碟。 

     ![](./media/data-box-gateway-deploy-provision-vmware/image14.png)

    向下捲動直到您看到 [新增硬碟]，展開它以檢視設定。 將 [虛擬裝置節點] 設為 [IDE 控制器 0]。 按 [下一步] 。

     ![](./media/data-box-gateway-deploy-provision-vmware/image15.png)

27. 在 [準備完成]  頁面上，檢閱與新的虛擬機器相關的所有設定。 確認 CPU 是 4、記憶體是 8192 MB、網路介面是 1，以及硬碟 2 具有 IDE 控制器 0。 按一下 [完成] 。 
   
    ![](./media/data-box-gateway-deploy-provision-vmware/image16.png)
    ![](./media/data-box-gateway-deploy-provision-vmware/image17.png)

您的虛擬機器已成功佈建。 您會看到效果通知，表示新的虛擬機器已新增至 VM 清單。 

![](./media/data-box-gateway-deploy-provision-vmware/image17.png)

下一個步驟是啟動該虛擬機器，然後取得 IP 位址。

> [!NOTE]
> 我們建議您不要在虛擬裝置上安裝 VMware 工具 (如同上面所佈建)。 安裝 VMware 工具將導致不支援的組態。

## <a name="start-the-virtual-device-and-get-the-ip"></a>啟動虛擬裝置並取得 IP 位址

請執行下列步驟來啟動您的虛擬裝置，並連線到該虛擬裝置。

#### <a name="to-start-the-virtual-device"></a>啟動虛擬裝置
1. 啟動虛擬裝置。 在右側窗格中，從 VM 清單選取您的裝置，並按一下滑鼠右鍵來顯示內容功能表。 選取 [電源]，然後選取 [開啟電源]。 此時您的虛擬機器應該會開機。 您可以在 Web 用戶端的窗格底部檢視狀態。

    ![](./media/data-box-gateway-deploy-provision-vmware/image19.png)

2. 再次選取您的 VM。 以滑鼠右鍵按一下並選取 [主控台]，然後選取 [在新視窗開啟]。

    ![](./media/data-box-gateway-deploy-provision-vmware/image20.png)

3. 虛擬機器主控台會在新視窗中開啟。 

    ![](./media/data-box-gateway-deploy-provision-vmware/image21.png)

4. 一旦裝置執行，將游標指向主控台視窗中間上方的索引標籤，並且按一下。 選取 [客體 OS > 傳送金鑰 > Ctrl+Alt+Delete]。 這樣會解除鎖定 VM。

   ![](./media/data-box-gateway-deploy-provision-vmware/image22.png)

5. 提供密碼以登入電腦。 預設密碼為 Password1。

   ![](./media/data-box-gateway-deploy-provision-vmware/image23.png)

6. 步驟 5 至 7 僅適用於在非 DHCP 環境中開機的情況。 如果您是在 DHCP 環境中，請略過這些步驟並前往步驟 8。 如果您是在非 DHCP 環境中將裝置開機，則會看到效果通知：**使用 Set-HcsIPAddress Cmdlet 以設定網路**。 
   
7. 若要設定網路，請在命令提示字元中使用 `Get-HcsIpAddress` 命令來列出虛擬裝置上已啟用的網路介面。 如果您的裝置有已啟用的單一網路介面，系統指派給該介面的預設名稱會是 `Ethernet`。

8. 使用 `Set-HcsIpAddress` Cmdlet 來設定網路。 範例如下所示：

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

9. 初始安裝程序完成，且裝置已開機之後，您將會看到裝置橫幅文字。 請記下 IP 位址及橫幅文字中的 URL，以便管理裝置。 您將使用此 IP 位址連線到虛擬裝置的 Web UI，以及完成本機安裝和啟用。

   ![](./media/data-box-gateway-deploy-provision-vmware/image24.png)

如果裝置不符合最低設定需求，橫幅文字中會出現錯誤訊息 (如下所示)。 您必須修改裝置設定，讓裝置有足夠的資源來符合最低需求。 然後您就可以將裝置重新啟動，並連線到該裝置。 請參閱[確認主機系統符合最低虛擬裝置需求](#check-the-host-system)中的最低組態需求。

<!---If you face any other error during the initial configuration using the local web UI, refer to the following workflows:

* Run diagnostic tests to [troubleshoot web UI setup](storsimple-ova-web-ui-admin.md#troubleshoot-web-ui-setup-errors).
* [Generate log package and view log files](storsimple-ova-web-ui-admin.md#generate-a-log-package).-->

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解資料箱閘道的相關主題，像是：

> [!div class="checklist"]
> * 確定主機符合最低裝置需求
> * 在 VMware 中佈建虛擬裝置
> * 啟動虛擬裝置，並取得 IP 位址

前進至下一個教學課程，以了解如何連線、設定及啟動您的虛擬裝置。

* [設定並連線到您資料箱閘道上的共用](data-box-gateway-deploy-connect-setup-activate.md)

