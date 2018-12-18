---
title: 部署 StorSimple 裝置管理員服務 | Microsoft Docs
description: 說明如何在 Azure 入口網站中建立和刪除 StorSimple 裝置管理員服務，並說明如何管理服務註冊金鑰。
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 28499494-8c4d-4a7f-9d44-94b2b8a93c93
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/29/2016
ms.author: alkohli
ms.openlocfilehash: 1881a0625b107ae1a90e5b772f5296a4d728973d
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38688148"
---
# <a name="deploy-the-storsimple-device-manager-service-for-storsimple-virtual-array"></a>部署 StorSimple Virtual Array 的 StorSimple 裝置管理員服務
## <a name="overview"></a>概觀

StorSimple 裝置管理員服務在 Microsoft Azure 中執行，並連接至多個 StorSimple 裝置。 建立服務之後，您可以使用服務從在瀏覽器中執行的 Microsoft Azure 入口網站管理這些裝置。 這可讓您從單一集中位置監視所有連接至 StorSimple 裝置管理員服務的裝置，因而將管理負擔降到最低。

StorSimple 裝置管理員服務相關的一般工作包括︰

* 建立服務
* 刪除服務
* 取得服務註冊金鑰。
* 重新產生服務註冊金鑰

本教學課程說明如何執行上述每一項工作。 本文所含資訊僅適用於 StorSimple Virtual Array。 如需 StorSimple 8000 系列的詳細資訊，請移至 [部署 StorSimple Manager 服務](storsimple-manage-service.md)。

## <a name="create-a-service"></a>建立服務

若要建立服務，您需要：

* 具有企業合約的訂用帳戶
* 使用中的 Microsoft Azure 儲存體帳戶
* 用於存取管理的計費資訊

您在建立服務時也可以選擇產生儲存體帳戶。

單一服務可以管理多個裝置。 不過，裝置不能跨越多個服務。 大型企業可以擁有多個服務執行個體使用不同的訂用帳戶、組織或甚至是部署位置。

> [!NOTE]
> 您需要個別的 StorSimple 裝置管理員服務執行個體來管理 StorSimple 8000 系列裝置和 StorSimple Virtual Array。


執行下列步驟來建立服務。

[!INCLUDE [storsimple-virtual-array-create-new-service](../../includes/storsimple-virtual-array-create-new-service.md)]

## <a name="delete-a-service"></a>刪除服務

刪除服務之前，請確定沒有任何連接的裝置正在使用它。 如果服務正在使用中，請停用連接的裝置。 停用作業將會斷絕裝置與服務之間的連接，但是會保留雲端中的裝置資料。

> [!IMPORTANT]
> 刪除服務之後，就無法回復此作業。 使用服務的任何裝置在可以搭配另一個服務使用之前，都必須恢復出廠預設值。 在此案例中，裝置上的本機資料以及組態將會遺失。
 

執行下列步驟來刪除服務。

#### <a name="to-delete-a-service"></a>刪除服務

1. 移至 [所有資源]。 搜尋您的 StorSimple 裝置管理員服務。 選取您想要刪除的服務。
   
    ![選取要刪除的服務](./media/storsimple-virtual-array-manage-service/deleteservice2.png)
2. 移至服務儀表板，確保沒有裝置連接至此服務。 如果沒有裝置向此服務註冊，您也會看到相關的橫幅訊息。 按一下 [刪除] 。
   
    ![刪除服務](./media/storsimple-virtual-array-manage-service/deleteservice3.png)

3. 當系統提示您確認時，按一下確認通知中的 [是]。 
   
    ![確認刪除服務](./media/storsimple-virtual-array-manage-service/deleteservice4.png)
4. 刪除服務可能需要數分鐘的時間。 成功刪除服務之後會通知您。
   
    ![成功刪除服務](./media/storsimple-virtual-array-manage-service/deleteservice6.png)

將會重新整理服務清單。

 ![更新的服務清單](./media/storsimple-virtual-array-manage-service/deleteservice7.png)

## <a name="get-the-service-registration-key"></a>取得服務註冊金鑰。
在您成功建立服務之後，您必須在 StorSimple 裝置註冊服務。 若要註冊您的第一個 StorSimple 裝置，您必須使用服務註冊金鑰。 若要使用現有的 StorSimple 服務註冊其他裝置，您需要註冊金鑰和服務資料加密金鑰 (在註冊期間於第一個裝置上產生)。 如需服務資料加密金鑰的詳細資訊，請參閱 [StorSimple 安全性](storsimple-security.md)。 您可以存取服務的 [金鑰] 刀鋒視窗來取得註冊金鑰。

執行下列步驟以取得服務註冊金鑰。

#### <a name="to-get-the-service-registration-key"></a>若要取得服務註冊金鑰
1. 在 [StorSimple 裝置管理員]中，移至 [管理]**&gt;**[金鑰]。
   
   ![[金鑰] 刀鋒視窗](./media/storsimple-virtual-array-manage-service/getregkey2.png)
2. [金鑰] 刀鋒視窗中會顯示服務註冊金鑰。 使用複製圖示來複製註冊金鑰。 

將服務註冊金鑰保存在安全的位置。 您需要這個金鑰，以及服務資料加密金鑰，才能對額外裝置註冊此服務。 取得服務註冊金鑰之後，您必須透過 Windows PowerShell for StorSimple 介面設定裝置。

## <a name="regenerate-the-service-registration-key"></a>重新產生服務註冊金鑰
如果您必須執行金鑰替換或服務系統管理員的清單已變更，則您必須重新產生服務註冊金鑰。 當您重新產生金鑰時，新的金鑰僅用於註冊後續裝置。 已註冊的裝置不會受到此程序影響。

執行下列步驟以重新產生服務註冊金鑰。

#### <a name="to-regenerate-the-service-registration-key"></a>重新產生服務註冊金鑰
1. 在 [StorSimple 裝置管理員]中，移至 [管理]**&gt;**[金鑰]。
   
   ![[金鑰] 刀鋒視窗](./media/storsimple-virtual-array-manage-service/getregkey2.png)
2. 在 [金鑰] 刀鋒視窗中，按一下 [重新產生]。
   
   ![按一下重新產生](./media/storsimple-virtual-array-manage-service/getregkey5.png)
3. 在 [重新產生服務註冊金鑰] 刀鋒視窗中，檢閱重新產生金鑰時所需的動作。 向此服務註冊的所有後續裝置會使用新的註冊金鑰。 按一下 [重新產生] 以確認。 註冊完成之後會通知您。
   
   ![確認重新產生金鑰](./media/storsimple-virtual-array-manage-service/getregkey3.png)
4. 新的服務註冊金鑰隨即顯示。
   
    ![確認重新產生金鑰](./media/storsimple-virtual-array-manage-service/getregkey4.png)
   
   複製這個金鑰並儲存，以對任何新的裝置註冊此服務。

## <a name="next-steps"></a>後續步驟
* 深入了解[開始使用](storsimple-virtual-array-deploy1-portal-prep.md) StorSimple Virtual Array。
* 深入了解 [管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

