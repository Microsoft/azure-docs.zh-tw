---
title: 管理 Microsoft Azure StorSimple 8000 系列裝置的 StorSimple 儲存體帳戶認證 | Microsoft Docs
description: 說明如何使用 StorSimple 裝置管理員的 [設定] 頁面來新增、編輯、刪除或替換儲存體帳戶的安全性金鑰。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/29/2017
ms.author: alkohli
ms.openlocfilehash: afaa4c090508ee5a2306c2be1515076e6bd4f100
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225720"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-your-storage-account-credentials"></a>使用 StorSimple 裝置管理員服務來管理儲存體帳戶認證

## <a name="overview"></a>概觀

StorSimple 裝置管理員服務的刀鋒視窗中，[設定] 區段會顯示所有可在 StorSimple 裝置管理員服務中建立的全域服務參數。 這些參數可以套用到與該服務連線的所有裝置，還包括：

* 儲存體帳戶認證
* 頻寬範本 
* 存取控制記錄 

此教學課程說明如何新增、編輯或刪除儲存體帳戶認證，或替換儲存體帳戶的安全性金鑰。

 ![儲存體帳戶認證的清單](./media/storsimple-8000-manage-storage-accounts/createnewstorageacct6.png)  

儲存體帳戶包含的認證可供 StorSimple 裝置用來存取雲端服務提供者的儲存體帳戶。 對於 Microsoft Azure 儲存體帳戶，像是帳戶名稱與主要存取金鑰就屬於這些認證。 

在 [儲存體帳戶認證] 刀鋒視窗上，為訂用帳戶計費而建立的所有儲存體帳戶都會以表格顯示，其中包含下列資訊：

* **名稱** – 帳戶建立時獲指派的唯一名稱。
* **啟用 SSL** – 是否已啟用 SSL 並透過安全通道進行裝置對雲端的通訊。
* **使用者** – 使用該儲存體帳戶的的磁碟區數目。

與儲存體帳戶相關可執行的最常見工作如下：

* 新增儲存體帳戶 
* 編輯儲存體帳戶 
* 刪除儲存體帳戶 
* 替換儲存體帳戶的金鑰 

## <a name="types-of-storage-accounts"></a>儲存體帳戶類型

有三種儲存體帳戶類型能與 StorSimple 裝置搭配使用。

* **自動產生的儲存體帳戶** – 正如其名，這類型的儲存體帳戶是在初次建立服務時自動產生。 若要深入了解如何建立此儲存體帳戶，請參閱[部署您的內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)中的[步驟 1：建立新的服務](storsimple-8000-deployment-walkthrough-u2.md#step-1-create-a-new-service)。 
* **服務訂用帳戶中的儲存體帳戶** – 這些是與相同服務訂用帳戶相關聯的 Azure 儲存體帳戶。 若要深入了解如何建立這些儲存體帳戶，請參閱 [關於 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)。 
* **服務訂用帳戶外的儲存體帳戶** – 這些是與服務毫無關聯的 Azure 儲存體帳戶，而且可能在服務建立之前便已存在 。

## <a name="add-a-storage-account"></a>新增儲存體帳戶

您可以提供唯一的易記名稱以及與儲存體帳戶(搭配指定的雲端服務提供者) 連結的存取認證來新增儲存體帳戶。 您也能選擇啟用安全通訊端層 (SSL) 模式，建立裝置與雲端之間網路通訊的安全通道。

您可以為指定的雲端服務提供者建立多個帳戶。 不過，請注意，建立儲存體帳戶之後，您無法變更雲端服務提供者。

儲存體帳戶在儲存時，服務會嘗試與您的雲端服務提供者通訊。 此時會驗證您提供的認證與存取資料。 只有當驗證成功時，才會建立儲存體帳戶。 如果驗證失敗，則會顯示適當的錯誤訊息。

使用下列程序來新增 Azure 儲存體帳戶認證︰

* 若要新增儲存體帳戶認證，而且其 Azure 訂用帳戶與裝置管理員服務相同
* 若要新增裝置管理員服務訂用帳戶外的 Azure 儲存體帳戶認證

[!INCLUDE [add-a-storage-account-update2](../../includes/storsimple-8000-configure-new-storage-account-u2.md)]

#### <a name="to-add-an-azure-storage-account-credential-outside-of-the-storsimple-device-manager-service-subscription"></a>若要新增 StorSimple 裝置管理員服務訂用帳戶外的 Azure 儲存體帳戶認證

1. 請瀏覽至您的 StorSimple 裝置管理員服務，選取它並按兩下。 這會開啟 [概觀] 刀鋒視窗。
2. 在 [設定] 區段內選取 [儲存體帳戶認證]。 這樣會列出與 StorSimple 裝置管理員服務相關聯的任何現有儲存體帳戶認證。
3. 按一下 [新增] 。
4. 在 [新增儲存體帳戶認證] 刀鋒視窗中，執行下列步驟︰
   
    1. 在 [訂用帳戶] 中，選取 [其他]。
   
    2. 提供 Azure 儲存體帳戶認證的名稱。
   
    3. 在 [儲存體帳戶存取金鑰]文字方塊中，提供 Azure 儲存體帳戶認證的主要「存取金鑰」。 若要取得此金鑰，請移至 Azure 儲存體服務，選取您的儲存體帳戶認證，然後按一下 [管理帳戶金鑰]。 現在，您可以複製主要存取金鑰。
   
    4. 若要啟用 SSL，請按一下 [啟用] 按鈕，為 StorSimple 裝置管理員服務與雲端之間的網路通訊建立安全通道。 只有當您在私人雲端內操作時，才按一下 [停用] 按鈕。
   
    5. 按一下 [新增] 。 成功建立儲存體帳戶認證之後會通知您。

5. 新建立的儲存體帳戶認證會顯示在 [StorSimple 設定裝置管理員服務] 刀鋒視窗的 [儲存體帳戶認證] 下方。
   


## <a name="edit-a-storage-account"></a>編輯儲存體帳戶

您可以編輯磁碟區容器所使用的儲存體帳戶。 如果您編輯的儲存體帳戶目前正在使用中，唯一可修改的欄位就是儲存體帳戶的存取金鑰。 您可以提供新的儲存體存取金鑰，並儲存更新的設定。

#### <a name="to-edit-a-storage-account"></a>若要編輯儲存體帳戶

1. 移至您的 StorSimple 裝置管理員服務。 在 [設定] 區段中，按一下 [儲存體帳戶認證]。

    ![儲存體帳戶認證](./media/storsimple-8000-manage-storage-accounts/editstorageacct1.png)

2. 在 [儲存體帳戶認證] 刀鋒視窗中，從儲存體帳戶認證清單中，選取並按一下您要編輯的項目。 

3. 您可以修改 [啟用 SSL] 選項。 您也可以按一下 [更多]，然後選取 [同步存取金鑰] 以替換您的儲存體帳戶存取金鑰。 如需如何執行金鑰替換的詳細資訊，請參閱[儲存體帳戶的金鑰替換](#key-rotation-of-storage-accounts)。 修改設定之後，按一下 [儲存]。 

    ![儲存編輯好的儲存體帳戶認證](./media/storsimple-8000-manage-storage-accounts/editstorageacct3.png)

4. 系統提示您進行確認時，按一下 [是] 。 

    ![確認修改](./media/storsimple-8000-manage-storage-accounts/editstorageacct4.png)

儲存體帳戶的設定將會更新並儲存。 

## <a name="delete-a-storage-account"></a>刪除儲存體帳戶

> [!IMPORTANT]
> 只有在其未由磁碟區容器使用時，您才可以刪除儲存體帳戶。 如果磁碟區容器正在使用儲存體帳戶，請先刪除磁碟區容器，然後再刪除相關聯的儲存體帳戶。

#### <a name="to-delete-a-storage-account"></a>若要刪除儲存體帳戶

1. 移至您的 StorSimple 裝置管理員服務。 在 [設定] 區段中，按一下 [儲存體帳戶認證]。

2. 在儲存體帳戶的表格式清單中，將滑鼠停留在您想要刪除的帳戶上方。 以滑鼠右鍵按一下以叫用操作功能表，然後按一下 [刪除]。

    ![刪除儲存體帳戶認證](./media/storsimple-8000-manage-storage-accounts/deletestorageacct1.png)

3. 當系統提示您確認時，按一下 [是]  繼續進行刪除。 表格式清單會更新以反映所做的變更。

    ![Confirm delete](./media/storsimple-8000-manage-storage-accounts/deletestorageacct2.png)

## <a name="key-rotation-of-storage-accounts"></a>替換儲存體帳戶的金鑰

基於安全性理由，通常是在資料中心內才需要替換金鑰。 每個 Microsoft Azure 訂用帳戶可以有一或多個相關聯的儲存體帳戶。 訂用帳戶與每個儲存體帳戶的存取金鑰可以控制這些帳戶的存取權。 

當您建立儲存體帳戶時，Azure 會產生兩個 512 位元的儲存體存取金鑰，可在存取儲存體帳戶時用於驗證。 有這兩個儲存體存取金鑰，您不需要中斷儲存體服務或對該服務的存取，就能重新產生金鑰。 目前使用中的金鑰是「主要」金鑰，而備份金鑰則稱為「次要」金鑰。 當 Microsoft Azure StorSimple 裝置存取您的雲端儲存體服務提供者時必須提供這些兩個機碼之一。

## <a name="what-is-key-rotation"></a>什麼是替換金鑰？

一般而言，應用程式只使用其中一個金鑰來存取您的資料。 經過一段時間之後，您可以讓應用程式切換為使用第二個金鑰。 在您將應用程式切換至次要金鑰之後，可以淘汰第一個金鑰，然後產生新的金鑰。 這種使用兩個金鑰的方式可讓您的應用程式存取資料，卻不會產生任何停機時間。

儲存體帳戶金鑰一律以加密的格式儲存在服務中。 不過，您可以透過 StorSimple 裝置管理員服務來重設。 服務可為相同訂用帳戶中的所有儲存體帳戶取得主要金鑰與次要金鑰，包括儲存體服務中建立的帳戶以及 StorSimple 裝置管理員服務初次建立時產生的預設儲存體帳戶。 StorSimple 裝置管理員服務將一律從 Azure 傳統入口網站取得這些金鑰，再以加密的方式儲存。

## <a name="rotation-workflow"></a>替換工作流程

Microsoft Azure 系統管理員可以直接存取儲存體帳戶 (透過 Microsoft Azure 儲存體服務) 來重新產生或變更主要金鑰或次要金鑰。 StorSimple 裝置管理員服務不會自動發現此變更。

若要向 StorSimple 裝置管理員服務通知此變更，您需要存取 StorSimple 裝置管理員服務，存取儲存體帳戶，然後同步處理主要金鑰或次要金鑰 (根據何者已變更而定)。 服務接著會取得最新的金鑰，將金鑰加密，然後將加密的金鑰傳送給裝置。

#### <a name="to-synchronize-keys-for-storage-accounts-in-the-same-subscription-as-the-service"></a>若要同步服務之相同訂用帳戶中的儲存體帳戶金鑰 
1. 移至您的 StorSimple 裝置管理員服務。 在 [設定] 區段中，按一下 [儲存體帳戶認證]。
2. 從儲存體帳戶的表格式清單中，按一下您要修改的項目。 

    ![同步處理金鑰](./media/storsimple-8000-manage-storage-accounts/syncaccesskey1.png)

3. 按一下 [更多]，然後選取 [同步存取金鑰] 以進行替換。   

    ![同步處理金鑰](./media/storsimple-8000-manage-storage-accounts/syncaccesskey2.png)

4. 在 StorSimple 裝置管理員服務中，您需要更新先前在 Microsoft Azure 儲存體服務中變更的金鑰。 如果主要存取金鑰有所變更 (已重新產生)，請選取 [主要] 金鑰。 如果次要金鑰有所變更，請選取 [次要] 金鑰。 按一下 [同步處理金鑰]。
      
      ![同步處理金鑰](./media/storsimple-8000-manage-storage-accounts/syncaccesskey3.png)

在成功同步金鑰之後，系統將會通知您。

#### <a name="to-synchronize-keys-for-storage-accounts-outside-of-the-service-subscription"></a>若要同步處理服務訂用帳戶外的儲存體帳戶金鑰
1. 在 [服務] 頁面上，按一下 [設定] 索引標籤。
2. 按一下 [新增/編輯儲存體帳戶] 。
3. 在對話方塊中，執行下列動作：
   
   1. 選取您要更新其存取金鑰的儲存體帳戶。
   2. 您必須更新 StorSimple 裝置管理員服務中的儲存體存取金鑰。 在此情況下，您可以看到儲存體存取金鑰。 在 [儲存體帳戶存取金鑰] 方塊中，輸入新的金鑰。 
   3. 儲存您的變更。 現在應已更新您的儲存體帳戶存取金鑰。

## <a name="next-steps"></a>後續步驟
* 深入了解 [StorSimple 安全性](storsimple-8000-security.md)。
* 深入了解[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

