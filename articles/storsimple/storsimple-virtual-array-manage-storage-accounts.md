---
title: 管理 StorSimple Virtual Array 儲存體帳戶認證 | Microsoft Docs
description: 針對與 StorSimple Virtual Array 相關聯的儲存體帳戶認證，說明如何使用 StorSimple 裝置管理員的 [設定] 頁面來新增、編輯、刪除或替換安全性金鑰。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 234bf8bb-d5fe-40be-9d25-721d7482bc3b
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.openlocfilehash: d2b6b7a5d2baf9e46cca18aec97f31beda1e2a38
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45736728"
---
# <a name="use-storsimple-device-manager-to-manage-storage-account-credentials-for-storsimple-virtual-array"></a>使用 StorSimple 裝置管理員來管理 StorSimple Virtual Array 的儲存體帳戶認證

## <a name="overview"></a>概觀
在 StorSimple Virtual Array 的 StorSimple 裝置管理員服務刀鋒視窗中，[設定] 區段會顯示可在 StorSimple Manager 服務中建立的全域服務參數。 這些參數可以套用到與該服務連線的所有裝置，還包括：

* 儲存體帳戶認證
* 存取控制記錄
  
  ![裝置管理員服務儀表板](./media/storsimple-virtual-array-manage-storage-accounts/ova-storageaccts-dashboard.png)  

本教學課程說明如何新增、編輯或刪除 StorSimple Virtual Array 的儲存體帳戶認證。 本教學課程中的資訊僅適用於 StorSimple Virtual Array。 如需如何在 8000 系列中管理儲存體帳戶的相關資訊，請參閱[使用 StorSimple Manager 服務來管理儲存體帳戶](storsimple-manage-storage-accounts.md)。

儲存體帳戶認證包含的認證可供裝置用來存取您在雲端服務提供者中的儲存體帳戶。 對於 Microsoft Azure 儲存體帳戶，像是帳戶名稱與主要存取金鑰就屬於這些認證。

在 [儲存體帳戶認證] 刀鋒視窗上，為訂用帳戶計費而建立的所有儲存體帳戶認證都會以表格顯示，其中包含下列資訊：

* **名稱** – 帳戶建立時獲指派的唯一名稱。
* **啟用 SSL** – 是否已啟用 SSL 並透過安全通道進行裝置對雲端的通訊。
  
  ![[設定] 區段](./media/storsimple-virtual-array-manage-storage-accounts/ova-storageaccountcredentials-blade.png)

在 [儲存體帳戶認證] 刀鋒視窗上，最常見可執行的儲存體帳戶認證相關工作如下︰

* 新增儲存體帳戶認證
* 編輯儲存體帳戶認證
* 刪除儲存體帳戶認證

## <a name="types-of-storage-account-credentials"></a>儲存體帳戶認證的類型
有三種儲存體帳戶認證類型可以與 StorSimple 裝置搭配使用。

* **自動產生的儲存體帳戶認證** – 正如其名，初次建立服務時會自動產生這種儲存體帳戶認證。 若要深入了解如何建立此儲存體帳戶認證，請參閱[建立新的服務](storsimple-virtual-array-manage-service.md#create-a-service)。
* **服務訂用帳戶中的儲存體帳戶認證** – 這些是與相同服務訂用帳戶相關聯的 Azure 儲存體帳戶認證。 若要深入了解如何建立這些儲存體帳戶認證，請參閱[關於 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)。
* **服務訂用帳戶外的儲存體帳戶認證** – 這些是與服務毫無關聯的 Azure 儲存體帳戶認證，可能在服務建立之前就存在。

## <a name="add-a-storage-account-credential"></a>新增儲存體帳戶認證
您可以提供唯一的易記名稱和連結至儲存體帳戶的存取認證，將儲存體帳戶認證新增至 StorSimple 裝置管理員服務設定。 您也能選擇啟用安全通訊端層 (SSL) 模式，建立裝置與雲端之間網路通訊的安全通道。

您可以為指定的雲端服務提供者建立多個帳戶。 當儲存體帳戶正在儲存時，服務會嘗試與您的雲端服務提供者通訊。 此時會驗證您提供的認證與存取資料。 只有當驗證成功時，才會建立儲存體帳戶認證。 如果驗證失敗，則會顯示適當的錯誤訊息。

使用下列程序來新增 Azure 儲存體帳戶認證︰

* 若要新增儲存體帳戶認證，而且其 Azure 訂用帳戶與裝置管理員服務相同
* 若要新增裝置管理員服務訂用帳戶外的 Azure 儲存體帳戶認證

#### <a name="to-add-a-storage-account-credential-that-has-the-same-azure-subscription-as-the-device-manager-service"></a>若要新增儲存體帳戶認證，而且其 Azure 訂用帳戶與裝置管理員服務相同

1. 瀏覽至您的裝置管理員服務，選取它並按兩下。 這會開啟 [概觀] 刀鋒視窗。
2. 在 [設定] 區段內選取 [儲存體帳戶認證]。
3. 按一下 [新增] 。
4. 在 [加入儲存體帳戶] 刀鋒視窗中，執行下列動作︰
   
    1. 在 [訂用帳戶] 中，選取 [目前]。
    2. 提供您 Azure 儲存體帳戶的名稱。
    3. 選取 [啟用]，為 StorSimple 裝置與雲端之間的網路通訊建立安全通道。 只有當您在私人雲端內操作時，才選取 [停用]。
    4. 按一下 [新增]。 成功建立儲存體帳戶之後會通知您。<br></br>
   
        ![新增現有儲存體帳戶認證](./media/storsimple-virtual-array-manage-storage-accounts/ova-add-storageacct.png)

#### <a name="to-add-an-azure-storage-account-credential-that-is-outside-of-the-device-manager-service-subscription"></a>若要新增裝置管理員服務訂用帳戶外的 Azure 儲存體帳戶認證

1. 瀏覽至您的裝置管理員服務，選取它並按兩下。 這會開啟 [概觀] 刀鋒視窗。
2. 在 [設定] 區段內選取 [儲存體帳戶認證]。 這樣會列出與 StorSimple 裝置管理員服務相關聯的任何現有儲存體帳戶認證。
3. 按一下 [新增] 。
4. 在 [加入儲存體帳戶] 刀鋒視窗中，執行下列動作︰
   
    1. 在 [訂用帳戶] 中，選取 [其他]。
   
    2. 提供 Azure 儲存體帳戶認證的名稱。
   
    3. 在 [儲存體帳戶存取金鑰]文字方塊中，提供 Azure 儲存體帳戶認證的主要「存取金鑰」。 若要取得此金鑰，請移至 Azure 儲存體服務，選取您的儲存體帳戶認證，然後按一下 [管理帳戶金鑰]。 現在，您可以複製主要存取金鑰。
   
    4. 若要啟用 SSL，請按一下 [啟用] 按鈕，為 StorSimple 裝置管理員服務與雲端之間的網路通訊建立安全通道。 只有當您在私人雲端內操作時，才按一下 [停用] 按鈕。
   
    5. 按一下 [新增] 。 成功建立儲存體帳戶認證之後會通知您。

5. 新建立的儲存體帳戶認證會顯示在 [StorSimple 設定裝置管理員服務] 刀鋒視窗的 [儲存體帳戶認證] 下方。
   
    ![新增裝置管理員服務訂用帳戶外的 Azure 儲存體帳戶認證](./media/storsimple-virtual-array-manage-storage-accounts/ova-add-outside-storageacct.png)

## <a name="edit-a-storage-account-credential"></a>編輯儲存體帳戶認證
您可以編輯裝置所使用的儲存體帳戶認證。 如果您編輯的儲存體帳戶認證目前正在使用中，您就只能修改該儲存體帳戶認證的存取金鑰和 SSL 模式欄位。 您可以提供新的儲存體存取金鑰，或是修改 [啟用 SSL 模式]  的選取項目，並儲存已更新的設定。

#### <a name="to-edit-a-storage-account-credential"></a>若要編輯儲存體帳戶認證
1. 瀏覽至您的裝置管理員服務，選取它並按兩下。 這會開啟 [概觀] 刀鋒視窗。
2. 在 [設定] 區段內選取 [儲存體帳戶認證]。 這樣會列出與 StorSimple 裝置管理員服務相關聯的任何現有儲存體帳戶認證。
3. 在儲存體帳戶認證的表格式清單中，選取並按兩下您想要修改的帳戶。
4. 在儲存體帳戶認證的 [屬性] 刀鋒視窗中，執行下列動作︰
   
   1. 如有必要，您可以修改 [啟用 SSL] 模式選項。
   2. 您可以選擇重新產生儲存體帳戶認證的存取金鑰。 如需詳細資訊，請參閱[重新產生儲存體帳戶金鑰](../storage/common/storage-account-manage.md#access-keys)。 提供新的儲存體帳戶認證金鑰。 而每個 Azure 儲存體帳戶，都有主要存取金鑰。
   3. 按一下 [屬性] 刀鋒視窗頂端的 [儲存] 以儲存設定。 [儲存體帳戶認證] 刀鋒視窗上會更新設定。
      
      ![編輯儲存體帳戶認證](./media/storsimple-virtual-array-manage-storage-accounts/ova-edit-storageacct.png)

## <a name="delete-a-storage-account-credential"></a>刪除儲存體帳戶認證
> [!IMPORTANT]
> 您只能刪除非使用中的儲存體帳戶認證。 如果您要刪除的儲存體帳戶認證正在使用中，系統會通知您。
> 
> 

#### <a name="to-delete-a-storage-account-credential"></a>若要刪除儲存體帳戶認證
1. 瀏覽至您的裝置管理員服務，選取它並按兩下。 這會開啟 [概觀] 刀鋒視窗。
2. 在 [設定] 區段內選取 [儲存體帳戶認證]。 這樣會列出與 StorSimple 裝置管理員服務相關聯的任何現有儲存體帳戶認證。
3. 在儲存體帳戶認證的表格式清單中，選取並按兩下您想要刪除的帳戶。
4. 在儲存體帳戶認證的 [屬性] 刀鋒視窗中，執行下列動作︰
   
   1. 按一下 [刪除]  來刪除認證。
   2. 當系統提示您確認時，按一下 [是]  繼續進行刪除。 表格式清單會更新以反映變更。
      
      ![刪除儲存體帳戶認證](./media/storsimple-virtual-array-manage-storage-accounts/ova-del-storageacct.png)

## <a name="synchronizing-storage-account-credential-keys"></a>同步處理儲存體帳戶認證金鑰
基於安全性理由，通常是在資料中心內才需要替換金鑰。 Microsoft Azure 系統管理員可以直接存取儲存體帳戶認證 (透過 Microsoft Azure 儲存體服務)，以重新產生或變更主要金鑰或次要金鑰。 StorSimple 裝置管理員服務不會自動發現這項變更。

若要向 StorSimple 裝置管理員服務通知此變更，您需要存取 StorSimple 裝置管理員服務，存取儲存體帳戶認證，然後同步處理主要金鑰或次要金鑰 (根據何者已變更而定)。 服務接著會取得最新的金鑰，將金鑰加密，然後將加密的金鑰傳送給裝置。

#### <a name="to-synchronize-keys-for-storage-account-credentials-in-the-same-subscription-as-the-service-azure-only"></a>若要在相同的服務訂用帳戶中同步處理儲存體帳戶認證的金鑰 (僅限 Azure)
1. 在服務登陸刀鋒視窗上，選取您的服務，按兩下服務名稱，然後在 [設定] 區段中按一下 [儲存體帳戶認證]。
2. 在 [儲存體帳戶認證] 刀鋒視窗的儲存體帳戶認證清單中，選取您想要同步處理金鑰的儲存體帳戶認證。
3. 在所選儲存體帳戶認證的 [屬性] 刀鋒視窗中，執行下列動作︰
   
    1. 按一下 [更多]，然後按一下 [同步存取金鑰]。
   
    2. 當系統提示您確認時，按一下 [同步金鑰] 完成同步處理。
    
4. 在 StorSimple 裝置管理員服務中，您需要更新先前在 Microsoft Azure 儲存體服務中變更的金鑰。 在 [同步儲存體帳戶金鑰] 刀鋒視窗中，如果主要存取金鑰已變更 (重新產生)，請按一下 [主要]，然後按一下 [同步金鑰]。 如果次要金鑰已變更，請按一下 [次要]，然後按一下 [同步金鑰]。
   
    ![同步存取金鑰](./media/storsimple-virtual-array-manage-storage-accounts/ova-sync-acess-key.png)

## <a name="next-steps"></a>後續步驟
* 了解如何 [管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

