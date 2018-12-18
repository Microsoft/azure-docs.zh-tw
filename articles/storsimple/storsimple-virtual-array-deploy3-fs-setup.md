---
title: 將 StorSimple Virtual Array 設定為檔案伺服器 | Microsoft Docs
description: 這是如何部署 StorSimple Virtual Array 的第三個教學課程，教導您如何將虛擬裝置設定成檔案伺服器。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: f609f6ff-0927-48bb-a68a-6d8985d2fe34
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/17/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: bf507fb21b314a6811db1c1e45a4356381caada1
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23111846"
---
# <a name="deploy-storsimple-virtual-array---set-up-as-file-server-via-azure-portal"></a>部署 StorSimple Virtual Array - 透過 Azure 入口網站設定為檔案伺服器
![](./media/storsimple-virtual-array-deploy3-fs-setup/fileserver4.png)

## <a name="introduction"></a>簡介
本文章說明如何執行初始安裝程序、為 StorSimple 檔案伺服器註冊、完成裝置安裝程序，以及建立並連接至 SMB 共用。 我們提供一系列如何將虛擬陣列完全部署為檔案伺服器或 iSCSI 伺服器的佈署教學課程，而這是該系列的最後一篇文章。

安裝及設定程序可能需要大約 10 分鐘的時間才能完成。 本文中的資訊僅適用於部署 StorSimple Virtual Array。 關於 StorSimple 8000 系列裝置的部署，請參閱︰[部署執行 Update 2 的 StorSimple 8000 系列裝置](storsimple-deployment-walkthrough-u2.md)。

## <a name="setup-prerequisites"></a>安裝的必要條件
在您設定及安裝 StorSimple Virtual Array 之前，請先確定：

* 您已根據[在 Hyper-V 中佈建 StorSimple Virtual Array](storsimple-virtual-array-deploy2-provision-hyperv.md) 或[在 VMware 中佈建 StorSimple Virtual Array](storsimple-virtual-array-deploy2-provision-vmware.md) 中的詳細說明，佈建並連接至虛擬陣列。
* 您已從您建立來管理 StorSimple Virtual Array 的 StorSimple 裝置管理員服務取得服務註冊金鑰。 如需詳細資訊，請參閱[步驟 2：取得 StorSimple Virtual Array 的服務註冊金鑰](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key)。
* 如果這是您要向現有 StorSimple 裝置管理員服務註冊的第二個或更後續的虛擬陣列，則您應該有服務資料加密金鑰。 當第一個裝置在此服務註冊成功時，這個金鑰就已經產生。 如果您遺失這個金鑰，請參閱 StorSimple Virtual Array 的 [取得服務資料加密金鑰](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) 。

## <a name="step-by-step-setup"></a>安裝的逐步指示
請使用下列逐步指示來安裝並設定您的 StorSimple Virtual Array。

## <a name="step-1-complete-the-local-web-ui-setup-and-register-your-device"></a>步驟 1：完成本機 Web UI 的安裝程序，並為裝置註冊
#### <a name="to-complete-the-setup-and-register-the-device"></a>如何完成裝置的安裝程序並為裝置註冊
1. 開啟瀏覽器視窗，並連接至本機 Web UI。 輸入：
   
   `https://<ip-address of network interface>`
   
   請使用您在上一個步驟記下的連線 URL。 您會看到錯誤指出網站的安全性憑證有問題。 請按一下 [繼續瀏覽此網頁] 。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image2.png)
2. 以 **StorSimpleAdmin** 身分登入虛擬陣列的 Web UI。 輸入您於[在 Hyper-V 中佈建 StorSimple Virtual Array](storsimple-virtual-array-deploy2-provision-hyperv.md) 或[在 VMware 中佈建 StorSimple Virtual Array](storsimple-virtual-array-deploy2-provision-vmware.md) 的「步驟 3：啟動虛擬陣列」中所變更的裝置系統管理員密碼。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image3.png)
3. 您將會前往 [首頁] 頁面。 此頁面說明設定虛擬陣列並向 StorSimple 裝置管理員服務註冊時所需的各種設定。 [網路設定]、[Web Proxy 設定] 及 [時間設定] 是可省略的。 只有 [裝置設定] 及 [雲端設定] 是必要的設定。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image4.png)
4. 在 [網路設定] 頁面的 [網路介面] 下方，系統會自動為您設定 DATA 0。 每個網路介面都預設會自動取得 IP 位址 (DHCP)。 因此，系統會自動指派 IP 位址、子網路和閘道 (IPv4 和 IPv6 皆適用)。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image5.png)
   
   如果您在佈建裝置的過程中新增了不只一個網路介面，您可以在此設定這些網路介面。 請注意，您可以將您的網路介面設定為僅 IPv4 或 IPv4 和 IPv6。 不支援僅 IPv6 組態。
5. DNS 伺服器是必要的，因為在裝置嘗試與雲端儲存體服務提供者通訊時，或是在根據名稱來解析已設定為檔案伺服器的裝置時，就需要使用 DNS 伺服器。 在 [網路設定] 頁面的 [DNS 伺服器] 下方：
   
   1. 系統會自動設定主要及次要 DNS 伺服器。 如果您選擇設定靜態 IP 位址，就可以指定 DNS 伺服器。 為了達到高可用性，我們建議您設定主要及次要 DNS 伺服器。
   2. 按一下 [套用]，套用及驗證網路設定。
6. 在 [裝置設定]  頁面中：
   
   1. 為裝置指派唯一的 [名稱]  。 這個名稱可以有 1 至 15 個字元，且可以包含字母、數字和連字號。
   2. 對於您要建立的裝置 [類型]，按一下 [檔案伺服器] 圖示 ![](./media/storsimple-virtual-array-deploy3-fs-setup/image6.png)。 檔案伺服器可讓您建立共用資料夾。
   3. 由於您的裝置是檔案伺服器，您必須讓裝置加入某個網域。 輸入 [網域名稱] 。
   4. 按一下 [套用]。
7. 此時畫面會出現對話方塊。 請以指定格式輸入網域認證。 按一下核取圖示。 系統會驗證該網域認證。 如果認證不正確，畫面會出現錯誤訊息。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image7.png)
8. 按一下 [套用]。 這將會套用並驗證裝置設定。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image8.png)
   
   > [!NOTE]
   > 確定您的虛擬陣列位於它自己的 Active Directory 組織單位 (OU) 中，且沒有套用或繼承任何群組原則物件 (GPO)。 群組原則可能會在 StorSimple Virtual Array 上安裝應用程式，例如防毒軟體。 安裝其他軟體並不受支援，而且可能導致資料損毀。 
   > 
   > 
9. (可省略) 設定 Web Proxy 伺服器。 雖然 Web Proxy 設定是選用的，但請注意，如果您使用 Web Proxy，就只能在此處設定它。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image9.png)
   
   在 [Web Proxy]  頁面中：
   
   1. 以下列格式提供 **Web Proxy URL**：「http://&lt;host-IP 位址」或「FDQN&gt;:連接埠號碼」。 請注意，此處不支援 HTTPS URL。
   2. 將 [驗證] 指定為 [基本] 或 [無]。
   3. 如果要使用驗證功能，您也必須提供 [使用者名稱] 和 [密碼]。
   4. 按一下 [套用]。 這將會驗證並套用您設定的 Web Proxy 設定。
10. (可省略) 設定裝置的時間設定，例如時區，以及主要和次要 NTP 伺服器。 NTP 伺服器是必要的，因為您的裝置必須讓時間同步，才能與您的雲端服務提供者進行驗證。
    
    ![](./media/storsimple-virtual-array-deploy3-fs-setup/image10.png)
    
    在 [時間設定]  頁面中：
    
    1. 根據裝置部署的地理位置，從下拉式清單中選取 [時區]  。 裝置的預設時區是太平洋標準時間。 裝置將針對所有排程的操作使用這個時區。
    2. 指定裝置的 [主要 NTP 伺服器]  ，或是接受預設值 time.windows.com。請確定您的網路允許 NTP 流量從您的資料中心通過網際網路。
    3. (選擇性) 指定裝置的 [次要 NTP 伺服器]  。
    4. 按一下 [Apply (套用)] 。 這將會驗證並套用您設定的時間設定。
11. 設定裝置的雲端設定。 在此步驟中，您將會完成本機裝置設定，然後向您的 StorSimple 裝置管理員服務註冊裝置。
    
    1. 輸入您在 **步驟 2：取得服務註冊金鑰** 中取得的 StorSimple Virtual Array [服務註冊金鑰](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key) 。
    2. 如果這是您向此服務註冊的第一個裝置，則會出現**服務資料加密金鑰**。 複製這個金鑰，並將它儲存在安全的位置。 這個金鑰需要與服務註冊金鑰搭配使用，才能向 StorSimple 裝置管理員服務註冊其他裝置。 
       
       如果這不是您向此服務註冊的第一個裝置，您將必須提供服務資料加密金鑰。 如需詳細資訊，請參閱使用本機 Web UI 取得 [服務資料加密金鑰](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) 。
    3. 按一下 [註冊] 。 這將讓裝置重新啟動。 您可能需要等待 2 至 3 分鐘，裝置才會註冊成功。 裝置重新啟動之後，您將會看到登入頁面。
       
       ![](./media/storsimple-virtual-array-deploy3-fs-setup/image13.png)
12. 返回 Azure 入口網站。 移至 [所有資源]，搜尋您的 StorSimple 裝置管理員服務。
    
    ![](./media/storsimple-virtual-array-deploy3-fs-setup/searchdevicemanagerservice1.png) 
13. 在已篩選的清單中，選取您的 StorSimple 裝置管理員服務，然後瀏覽至 [管理] > [裝置]。 在 [裝置] 刀鋒視窗上，確認裝置已成功連接至服務，且狀態為 [準備好進行設定]。
    
    ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs2m.png)

## <a name="step-2-configure-the-device-as-file-server"></a>步驟 2︰將裝置設定為檔案伺服器
在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟，完成必要的裝置設定。

#### <a name="to-configure-the-device-as-file-server"></a>若要將裝置設定為檔案伺服器
1. 移至 StorSimple 裝置管理員服務，再移至 [管理] > [裝置]。 在 [裝置] 刀鋒視窗中，選取您剛建立的裝置。 此裝置會顯示為 [準備好進行設定]。
   
   ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs2m.png) 
2. 按一下裝置，您會看到橫幅訊息，指出裝置已準備好進行設定。
   
    ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs3m.png)
3. 按一下命令列上的 [設定]。 這會開啟 [設定] 刀鋒視窗。 在 [設定] 刀鋒視窗中，執行下列動作：
   
    1. 檔案伺服器名稱會自動填入。
    
    2. 請確定雲端儲存體加密已設定為 [已啟用]。 這將會所有傳送至雲端的資料加密。 
    
    3. 256 位元 AES 金鑰與使用者定義的金鑰搭配用來加密。 指定 32 個字元的金鑰，然後重新輸入金鑰來加以確認。 將金鑰記錄在金鑰管理應用程式中，供日後參考。
    
    4. 按一下 [進行必要的設定]，以指定要用於裝置的儲存體帳戶認證。 如果未設定儲存體帳戶認證，請按一下 [新增]。 **請確認您使用的儲存體帳戶支援區塊 Blob。不支援分頁 Blob。** 關於 [區塊 Blob 和分頁 Blob](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) 的其他資訊。
   
    ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs6m.png) 
4. 在 [新增儲存體帳戶認證] 刀鋒視窗中，執行下列動作︰ 

    1. 如果儲存體帳戶與服務位於相同的訂用帳戶中，請選擇目前的訂用帳戶。 如果儲存體帳戶在服務訂用帳戶外，請選擇其他。 
    
    2. 從下拉式清單中，選擇現有的儲存體帳戶。 
    
    3. 將會根據指定的儲存體帳戶來自動填入位置。 
    
    4. 啟用 SSL 以確保裝置與雲端之間的安全網路通訊通道。
    
    5. 按一下 [新增]，新增此儲存體帳戶認證。 
   
        ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs8m.png)

5. 成功建立儲存體帳戶認證之後，[設定] 刀鋒視窗將會更新，以顯示指定的儲存體帳戶認證。 按一下 [設定] 。
   
   ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs11m.png)
   
   您會看到正在建立檔案伺服器。 成功建立檔案伺服器之後會通知您。
   
   ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs13m.png)
   
   裝置狀態也會變更為 [線上]。
   
   ![設定檔案伺服器](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs14m.png)
   
   您可以繼續新增共用。

## <a name="step-3-add-a-share"></a>步驟 3：新增共用
請在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟來建立共用。

#### <a name="to-create-a-share"></a>如何建立共用
1. 選取您在上一個步驟中設定的檔案伺服器裝置，然後按一下 ... \(或按一下滑鼠右鍵)。 在操作功能表中，選取 [新增共用]。 或者，您也可以按一下裝置命令列上的 [+ 新增共用]。
   
   ![新增共用](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs15m.png)
2. 指定下列共用設定：

    1. 共用的唯一名稱。 該名稱必須為包含 3 至 127 個字元的字串。
    
    2. 共用的選擇性 [說明]。 說明將可協助識別共用的擁有者。
    
    3. 共用的 [類型]。 類型可以是 [階層式] 或 [固定在本機]，而預設選項是 [階層式]。 對於需要本機保證、低延遲，以及高效能的工作負載，請選取 [固定在本機]  共用。 對於所有其他資料，請選取 [階層式]  共用。
    固定在本機的共用會密集佈建，且會確保共用中的主要資料會保留在裝置上，不會溢出到雲端。 而另一方面，階層式共用則是以精簡方式佈建。 當您建立階層式共用時，10% 的空間會佈建在本機層上，而 90% 的空間會佈建在雲端中。 舉例來說，如果您佈建 1 TB 的磁碟區，當資料使用階層式共用時，其中 100 GB 會位於本機的空間，900 GB 會位於雲端。 然而這也代表，如果裝置已沒有可用的本機空間，您就無法佈建階層式共用。
   
    4. 在 [將預設完整權限設為] 欄位中，指派權限給存取此共用的使用者或群組。 請以下列格式指定使用者或使用者群組的名稱：*john@contoso.com*。 我們建議您利用使用者群組 (而非單一使用者)，來授予可存取這些共用的系統管理員權限。 當您在此指派權限之後，就可以使用 [檔案總管] 來修改這些權限。
   
    5. 按一下 [新增] 以建立共用。 
    
        ![新增共用](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs18m.png)
   
        正在建立共用時會通知您。
   
        ![新增共用](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs19m.png)
   
    使用指定的設定來建立共用之後，[共用] 刀鋒視窗將會更新以反映新的共用。 根據預設，共用會啟用監視及備份。
   
    ![新增共用](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs22m.png)

## <a name="step-4-connect-to-the-share"></a>步驟 4：連線至共用
您現在必須連接至您在上一個步驟所建立的一或多個共用。 請在連接至 StorSimple Virtual Array 的 Windows Server 主機上執行這些步驟。

#### <a name="to-connect-to-the-share"></a>如何連線至共用
1. 按下 ![](./media/storsimple-virtual-array-deploy3-fs-setup/image22.png) + R 鍵，然後在 [執行] 視窗中，指定 &#92;&#92;&lt;檔案伺服器名稱&gt; 做為路徑，「檔案伺服器名稱」要換成您指派給檔案伺服器的裝置名稱。 按一下 [SERVICEPRINCIPAL] 。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image23.png)
2. 這會開啟 [檔案總管]。 您應該會看到所建立的共用以資料夾形式呈現。 選取並按兩下某個共用 (資料夾)，就能檢視該共用的內容。
   
   ![](./media/storsimple-virtual-array-deploy3-fs-setup/image24.png)
3. 現在您可以把檔案新增到這些共用，然後進行備份。

## <a name="next-steps"></a>後續步驟
了解如何使用本機 Web UI 來 [管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

