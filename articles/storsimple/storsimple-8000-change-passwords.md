---
title: 變更 StorSimple 密碼 | Microsoft Docs
description: 描述如何使用 StorSimple 裝置管理員服務變更 StorSimple Snapshot Manager 與裝置系統管理員密碼。
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
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: 7762f8499c67672f0a2ffed99e98baea4c940fa0
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108166"
---
# <a name="use-the-storsimple-device-manager-service-to-change-your-storsimple-passwords"></a>使用 StorSimple 裝置管理員服務變更您的 StorSimple 密碼

## <a name="overview"></a>概觀
Azure 入口網站的 [裝置設定] 選項包含所有裝置參數，可讓您重新設定 StorSimple 裝置管理員服務所管理的 StorSimple 裝置。 本教學課程說明如何使用 [裝置設定] 下的 [安全性] 選項，變更您的裝置系統管理員或 StorSimple Snapshot Manager 密碼。

## <a name="change-the-device-administrator-password"></a>變更裝置系統管理員密碼
當您使用 Windows PowerShell 介面來存取 StorSimple 裝置時，需要輸入裝置系統管理員密碼。 向服務註冊第一個 StorSimple 裝置時，此介面的預設密碼為 *Password1*。 為了確保資料的安全性，您必須在註冊程序結束時變更此密碼。 若未變更此密碼，您就無法結束註冊程序。 如需詳細資訊，請參閱 [步驟 3：透過 Windows PowerShell for StorSimple 設定和註冊裝置](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple)。

之後可以透過 Azure 入口網站，變更在註冊期間先透過 Windows PowerShell 介面設定的密碼。 執行下列步驟來變更裝置系統管理員密碼。

#### <a name="to-change-the-device-administrator-password"></a>若要變更裝置系統管理員密碼：
1. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。

2. 從裝置的表格式清單中，選取並按一下您想要變更其密碼的裝置。

    ![](./media/storsimple-8000-change-passwords/changepwd1.png)

3. 在 [設定] 刀鋒視窗中，移至 [裝置設定] > [安全性]。

    ![](./media/storsimple-8000-change-passwords/changepwd2.png)

4. 在 [安全性設定] 刀鋒視窗中，按一下 [密碼]，以變更裝置系統管理員密碼。

    ![](./media/storsimple-8000-change-passwords/changepwd3.png)

5. 在 [密碼] 刀鋒視窗中，提供含有 8 到 15 個字元的系統管理員密碼。 密碼必須是 3 個以上大寫、小寫、數字和特殊字元的組合。

6. 確認密碼。

    ![](./media/storsimple-8000-change-passwords/changepwd4.png)

7. 按一下 [儲存]，當系統提示您進行確認時，按一下 [是]。

    ![](./media/storsimple-8000-change-passwords/changepwd6.png)

現在應該已更新裝置系統管理員密碼。 您可以使用此修改過的密碼來存取 Windows PowerShell 介面。

## <a name="set-the-storsimple-snapshot-manager-password"></a>設定 StorSimple Snapshot Manager 密碼
StorSimple Snapshot Manager 軟體位於您的 Windows 主機上，而且可讓系統管理員以本機和雲端快照的形式管理 StorSimple 裝置的備份。

當您在 StorSimple Snapshot Manager 中設定裝置時，系統將提示您提供裝置 IP 位址和密碼來驗證您的儲存裝置。

您可以透過 Azure 入口網站設定或變更 StorSimple Snapshot Manager 密碼。 執行下列步驟來設定或變更 StorSimple Snapshot Manager 密碼。

#### <a name="to-set-the-storsimple-snapshot-manager-password"></a>若要設定 StorSimple Snapshot Manager 密碼
1. 移至 StorSimple 裝置管理員服務，然後按一下 [裝置]。

2. 從裝置的表格式清單中，選取並按一下您想要設定或變更其 StorSimple Snapshot Manager 密碼的裝置。

     ![](./media/storsimple-8000-change-passwords/changepwd1.png)

3. 在 [設定] 刀鋒視窗中，移至 [裝置設定] > [安全性]。

     ![](./media/storsimple-8000-change-passwords/changepwd2.png)

4. 在 [安全性設定] 刀鋒視窗中，按一下 [密碼]，以設定或變更 StorSimple Snapshot Manager 密碼。

     ![](./media/storsimple-8000-change-passwords/changepwd3.png) 

5. 在 [密碼] 刀鋒視窗中，輸入 14 或 15 個字元的密碼。 請確定密碼包含 3 個以上大寫、小寫、數字和特殊字元的組合。

6. 確認密碼。

     ![](./media/storsimple-8000-change-passwords/changepwd5.png)

7. 按一下 [儲存]，當系統提示您進行確認時，按一下 [是]。

     ![](./media/storsimple-8000-change-passwords/changepwd6.png)

StorSimple Snapshot Manager 密碼現在應該已更新。

## <a name="next-steps"></a>後續步驟
* 深入了解 [StorSimple 安全性](storsimple-8000-security.md)。
* [深入了解修改您的裝置組態](storsimple-8000-modify-device-config.md)。
* 深入了解[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

