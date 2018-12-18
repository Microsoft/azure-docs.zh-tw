---
title: 使用 Azure 資料箱閘道傳輸資料 | Microsoft Docs
description: 了解如何新增及連線到資料箱閘道裝置上的共用。
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: alkohli
ms.openlocfilehash: 6220a794c10f43d562155adbad910694a233ec88
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166012"
---
# <a name="tutorial-transfer-data-with-azure-data-box-gateway-preview"></a>教學課程：使用 Azure 資料箱閘道傳輸資料 (預覽)


## <a name="introduction"></a>簡介

本文說明如何新增及連線到資料箱閘道上的共用。 一旦新增共用之後，資料箱閘道裝置就可將資料傳輸至 Azure。

此程序可能需要大約 10 分鐘才能完成。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 新增共用
> * 連線到共用

> [!IMPORTANT]
> - 資料箱閘道處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 
 
## <a name="prerequisites"></a>必要條件

您將共用新增至資料箱閘道之前，請確定：

* 您已根據[在 Hyper-V 中佈建資料箱閘道](data-box-gateway-deploy-provision-hyperv.md)或[在 VMware 中佈建資料箱閘道](data-box-gateway-deploy-provision-vmware.md)中的詳細說明，佈建虛擬裝置並加以連線。 

    虛擬裝置已根據[連線並啟動 Azure 資料箱閘道](data-box-gateway-deploy-connect-setup-activate.md)中所述加以啟動，並已可供建立共用並傳輸資料。


## <a name="add-a-share"></a>新增共用

請在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟來建立共用。

1. 返回 Azure 入口網站。 移至 [所有資源]，並搜尋資料箱閘道資源。
    
2. 在經過篩選的資源清單中，選取資料箱閘道資源，然後瀏覽至 [概觀]。 按一下裝置命令列上的 [+ 新增共用]。
   
   ![新增共用](./media/data-box-gateway-deploy-add-shares/click-add-share.png)

4. 在 [新增共用] 中，指定共用設定。 為共用提供唯一的名稱。 

   共用名稱只能包含數字、小寫字母和連字號。 共用名稱的長度必須介於 3 到 63 個字元之間，且開頭為字母或數字。 每個連字號前後都必須緊接非連字號的字元。
    
5. 選取共用的 [類型]。 類型可以是 SMB 或 NFS，並以 SMB 為預設值。 SMB 是 Windows 用戶端的標準，NFS 則用於 Linux 用戶端。 視您選擇 SMB 或 NFS 共用而定，所顯示的選項會有些許不同。 

6. 您必須提供共用所在的儲存體帳戶。 如果容器已不存在，則會使用共用名稱在儲存體帳戶中建立容器。 如果容器已存在，則會使用現有的容器。 
    
7. 從區塊 Blob、分頁 Blob 或檔案中選擇 [儲存體服務]。 所選擇的服務類型取決於您想要資料以哪一種格式存在 Azure 中。 例如，在本例中，我們想要資料以區塊 Blob 存在 Azure 中，因此，我們選取 [區塊 Blob]。 如果選擇 [分頁 Blob]，您必須確定資料對齊 512 個位元組。 請注意，VHDX 一律對齊 512 個位元組。
   
8. 此步驟取決於您要建立 SMB 還是 NFS 共用。 
     
    - **如果建立 SMB 共用** - 在 [完整權限本機使用者] 欄位中，從 [新建] 或 [使用現有的] 中擇一。 如果建立新的本機使用者，請提供 [使用者名稱]、[密碼]，然後 [確認密碼]。 這會將使用權限指派給本機使用者。 當您在此指派權限之後，就可以使用 [檔案總管] 來修改這些權限。
    
        ![新增 SMB 共用](./media/data-box-gateway-deploy-add-shares/add-share-smb-1.png)
        
        如果您針對此共用資料勾選 [僅允許讀取作業]，則您可以選擇指定 [唯讀使用者]。
        
    - **如果建立 NFS 共用** - 您需要針對允許存取共用的用戶端提供其 IP 位址。

        ![新增 NFS 共用](./media/data-box-gateway-deploy-add-shares/add-share-nfs-1.png)
   
9. 按一下 [建立] 來建立共用。 
    
    正在建立共用時會通知您。 使用指定的設定來建立共用之後，[共用] 刀鋒視窗將會更新，以反映新的共用。 
    
    ![更新的共用清單](./media/data-box-gateway-deploy-add-shares/updated-list-of-shares.png) 

## <a name="connect-to-the-share"></a>連線至共用

在連線到資料箱閘道的 Windows Server 用戶端上執行這些步驟，以連線到共用。


1. 開啟命令視窗。 在命令提示字元中，輸入：

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    出現提示時，請輸入共用的密碼。 此命令的輸出範例會出現在這裡。

    ```powershell
    Microsoft Windows [Version 18.8.16299.192) 
    (c) 2817 microsoft Corporation. All rights reserved . 
    
    C: \Users\GatewayUser>net use \\10.10.10.60\newtestuser /u:Tota11yNewUser 
    Enter the password for 'TotallyNewUser' to connect to '10.10.10.60' • 
    The command completed successfully. 
    
    C: \Users\GatewayUser>
    ```   


2. 按 Windows + R。在 [執行] 視窗中，指定 `\\<device IP address>`。 按一下 [確定]。 這會開啟 [檔案總管]。 您應該會看到所建立的共用以資料夾形式呈現。 選取並按兩下某個共用 (資料夾)，就能檢視該共用的內容。
 
    ![連線至 SMB 共用](./media/data-box-gateway-deploy-add-shares/connect-to-share2.png)-->

    資料產生時會寫入這些共用，且裝置會將資料推送至雲端。

### <a name="connect-to-an-nfs-share"></a>連線到 NFS 共用

在連線到資料箱 Edge 的 Linux 用戶端上執行這些步驟。

1. 確定用戶端已安裝 NFSv4 用戶端。 若要安裝 NFS 用戶端，請使用下列命令：

   `sudo apt-get install nfs-common`

    如需詳細資訊，請移至[安裝 NFSv4 用戶端](https://help.ubuntu.com/community/SettingUpNFSHowTo#NFSv4_client)。

2. 安裝 NFS 用戶端之後，使用下列命令來掛接您在資料箱閘道裝置上建立的 NFS 共用：

   `sudo mount <device IP>:/<NFS share on device> /home/username/<Folder on local Linux computer>`

    設定掛接之前，請確定已建立本機電腦上要作為掛接點的目錄，且請勿包含任何檔案或子資料夾。

    下列範例示範如何透過 NFS 連線到閘道裝置上的共用。 虛擬裝置 IP 為 `10.10.10.60`，共用 `mylinuxshare2` 會掛接在 ubuntuVM 上，掛接點為 `/home/databoxubuntuhost/gateway`。

    `sudo mount -t nfs 10.10.10.60:/mylinuxshare2 /home/databoxubuntuhost/gateway`

> [!NOTE] 
> 下列需要注意的事項適用於預覽版本︰
> - 在共用中建立檔案之後，不支援重新命名檔案。 
> - 從共用中刪除檔案並不會刪除儲存體帳戶中的項目。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解資料箱閘道的相關主題，像是：

> [!div class="checklist"]
> * 新增共用
> * 連線到共用


請繼續進行下一個教學課程，以了解如何管理資料箱閘道。

> [!div class="nextstepaction"]
> [使用本機 Web UI 來管理資料箱閘道](http://aka.ms/dbg-docs)


