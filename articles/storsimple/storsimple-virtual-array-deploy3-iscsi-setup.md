---
title: Microsoft Azure StorSimple Virtual Array iSCSI 伺服器設定 | Microsoft Docs
description: 說明如何執行初始安裝程序、為 StorSimple iSCSI 伺服器註冊，以及完成裝置安裝程序。
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 4db116d1-978b-48e8-b572-a719a8425dbc
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.openlocfilehash: 076df176d7cd40c009aea27004fe0f4415999c80
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23112276"
---
# <a name="deploy-storsimple-virtual-array--set-up-as-an-iscsi-server-via-azure-portal"></a>部署 StorSimple Virtual Array - 透過 Azure 入口網站設定為 iSCSI 伺服器

![iSCSI 安裝程序流程](./media/storsimple-virtual-array-deploy3-iscsi-setup/iscsi4.png)

## <a name="overview"></a>概觀

此部署教學課程適用於 Microsoft Azure StorSimple Virtual Array。 本教學課程說明如何執行初始設定、註冊 StorSimple iSCSI 伺服器、完成裝置設定，然後在設定為 iSCSI 伺服器的 StorSimple Virtual Array 上建立、掛接、初始化及格式化磁碟區。 

完成此處描述的程序需要大約 30 分鐘至 1 個小時。 這篇文章中的資訊僅適用於 StorSimple Virtual Array。

## <a name="setup-prerequisites"></a>安裝的必要條件

在您設定及安裝 StorSimple Virtual Array 之前，請先確定：

* 您已根據[部署 StorSimple Virtual Array：在 Hyper-V 中佈建虛擬陣列](storsimple-ova-deploy2-provision-hyperv.md)或[部署 StorSimple Virtual Array：在 VMware 中佈建虛擬陣列](storsimple-virtual-array-deploy2-provision-vmware.md)中的說明，佈建並連接至虛擬陣列。
* 您已從您建立來管理 StorSimple Virtual Array 的 StorSimple 裝置管理員服務取得服務註冊金鑰。 如需詳細資訊，請參閱[部署 StorSimple Virtual Array：準備入口網站](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key)中的**步驟 2：取得服務註冊金鑰**。
* 如果這是您要向現有 StorSimple 裝置管理員服務註冊的第二個或更後續的虛擬陣列，則您應該有服務資料加密金鑰。 當第一個裝置在此服務註冊成功時，這個金鑰就已經產生。 如果您遺失這個金鑰，請參閱《 **使用 Web UI 來管理您的 StorSimple Virtual Array** 》一文中的〈 [取得服務資料加密金鑰](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key)〉。

## <a name="step-by-step-setup"></a>安裝的逐步指示

請使用下列逐步指示來安裝並設定您的 StorSimple Virtual Array：

* [步驟 1：完成本機 Web UI 的安裝程序，並為裝置註冊](#step-1-complete-the-local-web-ui-setup-and-register-your-device)
* [步驟 2：完成必要的裝置安裝程序](#step-2-complete-the-required-device-setup)
* [步驟 3：新增磁碟區](#step-3-add-a-volume)
* [步驟 4：掛接、初始化及格式化磁碟區](#step-4-mount-initialize-and-format-a-volume)

## <a name="step-1-complete-the-local-web-ui-setup-and-register-your-device"></a>步驟 1：完成本機 Web UI 的安裝程序，並為裝置註冊

#### <a name="to-complete-the-setup-and-register-the-device"></a>如何完成裝置的安裝程序並為裝置註冊

1. 開啟瀏覽器視窗。 若要連接至 Web UI，請輸入︰
   
    `https://<ip-address of network interface>`
   
    請使用您在上一個步驟記下的連線 URL。 您會看到錯誤訊息，通知您網站的安全性憑證有問題。 請按一下 [繼續瀏覽此網頁] 。
   
    ![安全性憑證錯誤](./media/storsimple-virtual-array-deploy3-iscsi-setup/image3.png)
2. 以 **StorSimpleAdmin**的帳戶名稱登入虛擬裝置的 Web UI。 請輸入您在[部署 StorSimple Virtual Array - 在 Hyper-V 中佈建虛擬裝置](storsimple-virtual-array-deploy2-provision-hyperv.md)或[部署 StorSimple Virtual Array - 在 VMware 中佈建虛擬裝置](storsimple-virtual-array-deploy2-provision-vmware.md)的「步驟 3：啟動虛擬裝置」中所變更的裝置系統管理員密碼。
   
    ![登入頁面](./media/storsimple-virtual-array-deploy3-iscsi-setup/image4.png)
3. 您將會進入 [首頁]  頁面。 此頁面說明設定虛擬裝置並向 StorSimple 裝置管理員服務註冊時所需的各種設定。 請注意，[網路設定]、[Web Proxy 設定] 及 [時間設定] 是可省略的。 只有 [裝置設定] 及 [雲端設定] 是必要的設定。
   
    ![首頁](./media/storsimple-virtual-array-deploy3-iscsi-setup/image5.png)
4. 在 [網路設定] 頁面的 [網路介面] 下方，系統會自動為您設定 DATA 0。 每個網路介面都預設會自動取得 IP 位址 (DHCP)。 因此，系統會自動指派 IP 位址、子網路和閘道 (IPv4 和 IPv6 皆適用)。
   
    因為您計畫將裝置部署為 iSCSI 伺服器 (以佈建區塊儲存體)，我們建議您停用 [自動取得 IP 位址]  選項，並設定靜態 IP 位址。
   
    ![網路設定頁面](./media/storsimple-virtual-array-deploy3-iscsi-setup/image6.png)
   
    如果您在佈建裝置的過程中新增了不只一個網路介面，您可以在此設定這些網路介面。 請注意，您可以將您的網路介面設定為僅 IPv4 或 IPv4 和 IPv6。 不支援僅 IPv6 組態。
5. DNS 伺服器是必要的，因為在裝置嘗試與雲端儲存體服務提供者通訊時，或是在根據名稱來解析已設定為檔案伺服器的裝置時，就需要使用 DNS 伺服器。 在 [網路設定] 頁面的 [DNS 伺服器] 下方：
   
   1. 系統會自動設定主要及次要 DNS 伺服器。 如果您選擇設定靜態 IP 位址，就可以指定 DNS 伺服器。 為了達到高可用性，我們建議您設定主要及次要 DNS 伺服器。
   2. 按一下 [套用]。 這將會套用並驗證網路設定。
6. 在 [裝置設定]  頁面上：
   
   1. 為裝置指派唯一的 [名稱]  。 這個名稱可以有 1 至 15 個字元，且可以包含字母、數字和連字號。
   2. 對於您要建立的裝置 [類型]，按一下 [iSCSI 伺服器] 圖示 ![iSCSI 伺服器圖示](./media/storsimple-virtual-array-deploy3-iscsi-setup/image7.png)。 ISCSI 伺服器可讓您佈建區塊儲存體。
   3. 指定您是否想讓此裝置加入網域。 如果您的裝置是 iSCSI 伺服器，您可以省略加入網域這個步驟。 如果您決定不將 iSCSI 伺服器加入網域，請按一下 [套用] 並等待設定套用完畢，然後前往下一個的步驟。
      
       如果您想要讓裝置加入網域， 輸入 [網域名稱]，然後按一下 [套用]。
      
      > [!NOTE]
      > 如果將您的 iSCSI 加入網域，請確定您的虛擬陣列位於它自己的 Microsoft Azure Active Directory 組織單位 (OU) 中，且沒有套用群組原則物件 (GPO)。
      > 
      > 
   4. 此時畫面會出現對話方塊。 請以指定格式輸入網域認證。 按一下核取圖示  ![核取圖示](./media/storsimple-virtual-array-deploy3-iscsi-setup/image15.png)》一文中的指示來佈建虛擬裝置，並與該虛擬裝置連線。 系統將會驗證該網域認證。 如果認證不正確，畫面會出現錯誤訊息。
      
       ![認證](./media/storsimple-virtual-array-deploy3-iscsi-setup/image8.png)
   5. 按一下 [套用]。 這將會套用並驗證裝置設定。
7. (可省略) 設定 Web Proxy 伺服器。 雖然 Web Proxy 設定是選用的，但請注意，如果您使用 Web Proxy，就只能在此處設定它。
   
    ![設定 Web Proxy](./media/storsimple-virtual-array-deploy3-iscsi-setup/image9.png)
   
    在 [Web Proxy]  頁面上：
   
   1. 以下列格式提供 **Web Proxy URL**：「http://host-IP 位址」或「FDQN:連接埠號碼」。 請注意，此處不支援 HTTPS URL。
   2. 將 [驗證] 指定為 [基本] 或 [無]。
   3. 如果您要使用驗證功能，您也必須提供 [使用者名稱] 和 [密碼]。
   4. 按一下 [套用]。 這將會驗證並套用您設定的 Web Proxy 設定。
8. (可省略) 設定裝置的時間設定，例如時區，以及主要和次要 NTP 伺服器。 NTP 伺服器是必要的，因為您的裝置必須讓時間同步，才能與您的雲端服務提供者進行驗證。
   
    ![時間設定](./media/storsimple-virtual-array-deploy3-iscsi-setup/image10.png)
   
    在 [時間設定]  頁面上：
   
   1. 根據裝置部署的地理位置，從下拉式清單中選取 [時區]  。 裝置的預設時區是太平洋標準時間。 裝置將針對所有排程的操作使用這個時區。
   2. 指定裝置的 [主要 NTP 伺服器]  ，或是接受預設值 time.windows.com。請確定您的網路允許 NTP 流量從您的資料中心通過網際網路。
   3. (選擇性) 指定裝置的 [次要 NTP 伺服器]  。
   4. 按一下 [Apply (套用)] 。 這將會驗證並套用您設定的時間設定。
9. 設定裝置的雲端設定。 在此步驟中，您將會完成本機裝置設定，然後向您的 StorSimple 裝置管理員服務註冊裝置。
   
   1. 輸入您在[部署 StorSimple Virtual Array：準備入口網站](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key)的**步驟 2：取得服務註冊金鑰**中取得的**服務註冊金鑰**。
   2. 如果這不是您向此服務註冊的第一個裝置，您將必須提供「服務資料加密金鑰」 。 這個金鑰需要與服務註冊金鑰搭配使用，才能向 StorSimple 裝置管理員服務註冊其他裝置。 如需詳細資訊，請參閱使用本機 Web UI 來 [取得服務資料加密金鑰](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) 。
   3. 按一下 [註冊] 。 這將讓裝置重新啟動。 您可能需要等待 2 至 3 分鐘，裝置才會註冊成功。 裝置重新啟動之後，您將會看到登入頁面。
      
      ![註冊裝置](./media/storsimple-virtual-array-deploy3-iscsi-setup/image11.png)
10. 返回 Azure 入口網站。
11. 瀏覽至服務的 [裝置] 刀鋒視窗。 如果您有大量資源，請依序按一下 [所有資源]、您的服務名稱 (如有必要，請搜尋) 及 [裝置]。
12. 在 [裝置] 刀鋒視窗上，查閱狀態來確認裝置已成功連接至服務。 裝置狀態應該是 [準備好進行設定]。
    
    ![註冊裝置](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis1m.png)

## <a name="step-2-configure-the-device-as-iscsi-server"></a>步驟 2︰將裝置設定為 iSCSI 伺服器

在 Azure 入口網站中執行下列步驟，完成必要的裝置設定。

#### <a name="to-configure-the-device-as-iscsi-server"></a>若要將裝置設定為 iSCSI 伺服器

1. 移至 StorSimple 裝置管理員服務，再移至 [管理] > [裝置]。 在 [裝置] 刀鋒視窗中，選取您剛建立的裝置。 此裝置會顯示為 [準備好進行設定]。
   
    ![將裝置設定為 iSCSI 伺服器](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis1m.png) 
2. 按一下裝置，您會看到橫幅訊息，指出裝置已準備好進行設定。
   
    ![將裝置設定為 iSCSI 伺服器](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis2m.png)  
3. 按一下裝置命令列上的 [設定]。 這會開啟 [設定] 刀鋒視窗。 在 [設定] 刀鋒視窗中，執行下列動作：
   
   * iSCSI 伺服器名稱會自動填入。
   * 請確定雲端儲存體加密已設定為 [已啟用]。 這可確保從裝置傳送至雲端的資料已加密。
   * 指定 32 個字元的加密金鑰，並將它記錄在金鑰管理應用程式中，供日後參考。
   * 選取要與裝置搭配使用的儲存體帳戶。 在這個訂用帳戶中，您可以選取現有的儲存體帳戶，或按一下 [新增]，從不同的訂用帳戶中選擇帳戶。
     
     ![將裝置設定為 iSCSI 伺服器](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis4m.png)
4. 按一下 [設定] 來完成 iSCSI 伺服器的設定。
   
    ![將裝置設定為 iSCSI 伺服器](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis5m.png) 
5. 正在建立 iSCSI 伺服器時會通知您。 成功建立 iSCSI 伺服器之後，[裝置] 刀鋒視窗會更新，而且對應的裝置狀態為 [線上]。
   
    ![將裝置設定為 iSCSI 伺服器](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis9m.png)

## <a name="step-3-add-a-volume"></a>步驟 3：新增磁碟區

1. 在 [裝置] 刀鋒視窗中，選取您剛設定為 iSCSI 伺服器的裝置。 按一下 [...] \(或以滑鼠右鍵按一下此資料列)，然後從操作功能表中選取 [新增磁碟區]。 您也可以從命令列按一下 [+ 新增磁碟區]。 這會開啟 [新增磁碟區] 刀鋒視窗。
   
    ![新增磁碟區](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis10m.png)
2. 在 [新增磁碟區] 刀鋒視窗中，執行下列動作：
   
   * 在 [磁碟區名稱] 欄位中，輸入磁碟區的唯一名稱。 該名稱必須為包含 3 至 127 個字元的字串。
   * 在 [類型] 下拉式清單中，指定要建立 [階層式] 還是 [固定在本機] 磁碟區。 對於需要本機保證、低延遲及更高效能的工作負載，請選取 [固定在本機的磁碟區]。 針對所有其他資料，請選取 [階層式磁碟區]**磁碟區**。
   * 在 [容量] 欄位中，指定磁碟區大小。 階層式磁碟區必須介於 500 GB 和 5 TB 之間，而固定在本機的磁碟區必須介於 50 GB 和 500 GB 之間。
     
     固定在本機的磁碟區會密集佈建，且會確保磁碟區中的主要資料會保留在裝置上，不會溢出到雲端。
     
     反之，階層式磁碟區是精簡佈建。 當您建立階層式磁碟區時，大約 10% 的空間會佈建在本機層上，而 90% 的空間會佈建在雲端中。 舉例來說，如果您佈建 1 TB 的磁碟區，當資料使用階層式磁碟區時，其中 100 GB 會位於本機的空間，900 GB 會位於雲端。 然而這也代表，如果裝置已沒有可用的空間，您就無法佈建階層式共用 (因為那 10% 的空間將無法使用)。
     
     ![新增磁碟區](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis12.png)
   * 按一下 [連接的主機]，選取存取控制記錄 (ACR) (對應於您要連接至此磁碟區的 iSCSI 啟動器，然後按一下 [選取]。 <br><br> 
3. 若要新增連接的主機，請按一下 [新增]，輸入主機的名稱及其 iSCSI 合格名稱 (IQN)，然後按一下 [新增]。 如果您沒有 IQN，請前往 [附錄 A：取得 Windows Server 主機的 IQN](#appendix-a-get-the-iqn-of-a-windows-server-host)。
   
      ![新增磁碟區](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis15m.png)
4. 磁碟區設定完成之後，按一下 [確定]。 將會使用指定的設定來建立磁碟區，您會看到通知。 根據預設，磁碟區的監視及備份功能將會啟用。
   
     ![新增磁碟區](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis18m.png)
5. 若要確認已成功建立磁碟區，請移至 [磁碟區] 刀鋒視窗。 您應該會看到此處列出該磁碟區。
   
   ![新增磁碟區](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis20m.png)

## <a name="step-4-mount-initialize-and-format-a-volume"></a>步驟 4：掛接、初始化及格式化磁碟區

請執行下列步驟，以便在 Windows Server 主機上掛接、初始化及格式化您的 StorSimple 磁碟區。

#### <a name="to-mount-initialize-and-format-a-volume"></a>掛接、初始化及格式化磁碟區

1. 在適當的伺服器上開啟 [iSCSI 啟動器] 應用程式。
2. 在 [iSCSI 啟動器屬性] 視窗的 [探索] 索引標籤上，按一下 [探索入口網站]。
   
    ![探索入口網站](./media/storsimple-virtual-array-deploy3-iscsi-setup/image22.png)
3. 在 [探索目標入口網站] 對話方塊中，提供已啟用 iSCSI 網路介面的 IP 位址，然後按一下 [確定]。
   
    ![IP 位址](./media/storsimple-virtual-array-deploy3-iscsi-setup/image23.png)
4. 在 [iSCSI 啟動器屬性] 視窗的 [目標] 索引標籤上，找到 [探索到的目標]。 (每個磁碟區都會是探索到的目標。)裝置狀態應會顯示為 [非使用中] 。
   
    ![探索到的目標](./media/storsimple-virtual-array-deploy3-iscsi-setup/image24.png)
5. 選取目標裝置，然後按一下 [連接]。 連接裝置之後，狀態應會變更為 [已連接]。 如需有關如何使用 Microsoft iSCSI 啟動器的詳細資訊，請參閱[安裝和設定 Microsoft iSCSI 啟動器][1]。
   
    ![選取目標裝置](./media/storsimple-virtual-array-deploy3-iscsi-setup/image25.png)
6. 在 Windows 主機上按 Windows 標誌鍵 + X，然後按一下 [執行] 。
7. 在 [執行] 對話方塊中，輸入 **Diskmgmt.msc**。 按一下 [確定]，隨即會出現 [磁碟管理] 對話方塊。 右窗格將會顯示主機上的磁碟區。
8. 在 [磁碟管理]  視窗中，將會出現掛接的磁碟區，如下圖所示。 以滑鼠右鍵按一下探索到的磁碟區 (按一下磁碟名稱)，然後按一下 [線上] 。
   
    ![磁碟管理](./media/storsimple-virtual-array-deploy3-iscsi-setup/image26.png)
9. 按一下滑鼠右鍵，然後選取 [初始化磁碟] 。
   
    ![初始化磁碟 1](./media/storsimple-virtual-array-deploy3-iscsi-setup/image27.png)
10. 在對話方塊中，選取要初始化的磁碟，然後按一下 [確定] 。
    
    ![初始化磁碟 2](./media/storsimple-virtual-array-deploy3-iscsi-setup/image28.png)
11. [新增簡單磁碟區] 精靈會隨即啟動。 請選取磁碟大小，然後按 [下一步] 。
    
    ![新增磁碟區精靈 1](./media/storsimple-virtual-array-deploy3-iscsi-setup/image29.png)
12. 指派一個磁碟機代號給磁碟區，然後按 [下一步] 。
    
    ![新增磁碟區精靈 2](./media/storsimple-virtual-array-deploy3-iscsi-setup/image30.png)
13. 輸入要格式化磁碟區所需的參數。 **Windows Server 只支援 NTFS。** 將配置單位大小設定為 64K。 並提供您磁碟區的標籤。 建議的最佳做法是這個名稱與您在 StorSimple Virtual Array 上提供的磁碟區名稱相同。 按 [下一步] 。
    
    ![新增磁碟區精靈 3](./media/storsimple-virtual-array-deploy3-iscsi-setup/image31.png)
14. 查看您磁碟區的各個值，然後按一下 [完成] 。
    
    ![新增磁碟區精靈 4](./media/storsimple-virtual-array-deploy3-iscsi-setup/image32.png)
    
    該磁碟區將會在 [磁碟管理]  on the  頁面。
    
    ![磁碟區線上](./media/storsimple-virtual-array-deploy3-iscsi-setup/image33.png)

## <a name="next-steps"></a>後續步驟

了解如何使用本機 Web UI 來 [管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

## <a name="appendix-a-get-the-iqn-of-a-windows-server-host"></a>附錄 A：取得 Windows Server 主機的 IQN

請執行下列步驟，以取得正在執行 Windows Server 2012 之 Windows 主機的 iSCSI 限定名稱 (IQN)。

#### <a name="to-get-the-iqn-of-a-windows-host"></a>取得 Windows 主機的 IQN

1. 在 Windows 主機上啟動 Microsoft iSCSI 啟動器。
2. 在 [iSCSI 啟動器屬性] 視窗的 [設定] 索引標籤上，選取並複製 [啟動器名稱] 欄位的字串。
   
    ![iSCSI 啟動器屬性](./media/storsimple-virtual-array-deploy3-iscsi-setup/image34.png)
3. 儲存這個字串。

<!--Reference link-->
[1]: https://technet.microsoft.com/library/ee338480(WS.10).aspx



