---
title: 變更 StorSimple Virtual Array 裝置系統管理員密碼 | Microsoft Docs
description: 說明如何使用 Azure 入口網站或 StorSimple Virtual Array Web UI 來變更裝置系統管理員密碼。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 11490814-d9fd-4dc7-9c3b-55dd2c23eaf1
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 260a23003d705e6598da8c51bb5a96f2539a0014
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109426"
---
# <a name="change-the-storsimple-virtual-array-device-administrator-password-via-storsimple-device-manager"></a>透過 StorSimple 裝置管理員變更 StorSimple Virtual Array 裝置系統管理員密碼

## <a name="overview"></a>概觀

當您使用 Windows PowerShell 介面來存取 StorSimple Virtual Array 時，您需要輸入裝置系統管理員密碼。 StorSimple 裝置第一次佈建並啟動時，預設的密碼是 *Password1*。 為了資料的安全性，您第一次登入時預設密碼便會過期，且系統會要求您變更密碼。

裝置部署到生產環境之後，您隨時可以使用本機 Web UI 或 Azure 入口網站變更裝置系統管理員密碼。 本文章說明上述各程序。

 ![裝置刀鋒視窗](./media/storsimple-virtual-array-change-device-admin-password/ova-devices-blade.png)

## <a name="use-the-azure-portal-to-change-the-password"></a>使用 Azure 入口網站變更密碼

執行下列步驟來透過 Azure 入口網站變更裝置系統管理員密碼。

#### <a name="to-change-the-device-administrator-password-via-the-azure-portal"></a>透過 Azure 入口網站變更裝置系統管理員密碼

1. 在服務登陸頁面上，選取您的服務，按兩下服務名稱，然後按一下在 [管理] 區段內按一下 [裝置]。 這會開啟 [裝置] 刀鋒視窗來列出您的所有 StorSimple Virtual Array 裝置。

2. 在 [裝置] 刀鋒視窗中，按兩下需要變更密碼的裝置。

3. 在裝置的 [設定] 刀鋒視窗中，按一下 [安全性]。

4. 在 [安全性設定] 刀鋒視窗中，執行下列動作︰
   
   1. 向下捲動至 [ **裝置系統管理員密碼** ] 區段。 提供含有 8 到 15 個字元的系統管理員密碼。
   2. 確認密碼。
   3. 按一下刀鋒視窗頂端的 [儲存] 。

現在已變更裝置系統管理員密碼。 您可以使用修改的密碼在本機存取該裝置。

![安全性設定刀鋒視窗](./media/storsimple-virtual-array-change-device-admin-password/ova-change-device-pwd.png)

## <a name="use-the-local-web-ui-to-change-the-password"></a>使用本機 Web UI 來變更密碼

執行下列步驟來透過本機 Web UI 變更裝置系統管理員密碼。

#### <a name="to-change-the-device-administrator-password-via-the-local-web-ui"></a>透過本機 Web UI 變更裝置系統管理員密碼

1. 在本機 Web UI 中，對您的裝置按一下 [維護]  >  [變更密碼]。
   
    ![變更 password1](./media/storsimple-virtual-array-change-device-admin-password/image40.png)
2. 輸入 **目前的密碼**。
3. 提供 **新密碼**。 密碼長度必須至少為 8 個字元。 密碼必須包含下列 4 項的其中 3 項：大寫、小寫、數字和特殊字元。
   
    請注意，您的密碼不能與最近的 24 個密碼相同。
4. 請重新輸入密碼來加以確認。
   
    ![變更 password2](./media/storsimple-virtual-array-change-device-admin-password/image41.png)
5. 在頁面底部，按一下 [套用] 。 現在已套用新的密碼。 如果變更密碼未成功，您會看到下列錯誤：
   
    ![密碼錯誤](./media/storsimple-virtual-array-change-device-admin-password/image42.png)
   
    成功更新密碼之後會通知您。 您即可使用修改的密碼在本機存取該裝置。


## <a name="next-steps"></a>後續步驟
了解如何 [管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

