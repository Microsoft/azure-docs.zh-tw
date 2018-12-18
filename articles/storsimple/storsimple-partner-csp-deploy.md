---
title: Microsoft Azure StorSimple 和雲端解決方案提供者方案概觀 | Microsoft Docs
description: StorSimple 和適用於 StorSimple 合作夥伴的 CSP 概觀。
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
ms.date: 02/08/2017
ms.author: alkohli
ms.openlocfilehash: c8cb51093142146fc7d43b51a62d949f6cc38988
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109576"
---
# <a name="deploy-storsimple-virtual-array-for-cloud-solution-provider-program"></a>部署適用於雲端解決方案提供者方案的 StorSimple Virtual Array

## <a name="overview"></a>概觀

雲端解決方案提供者 (CSP) 合作夥伴可以為客戶部署 StorSimple Virtual Array。 CSP 合作夥伴可以建立 StorSimple 裝置管理員服務。 然後，此服務可用來部署及管理 StorSimple Virtual Array，以及相關聯的共用、磁碟區和備份。

本文說明 CSP 合作夥伴如何新增客戶，或將新的訂用帳戶新增至現有的客戶，然後建立服務在 CSP 中部署 StorSimple Virtual Array。

## <a name="prerequisites"></a>先決條件

開始之前，請確定︰

- 您已註冊 CSP 方案。
- 您具備有效的[合作夥伴中心](http://partnercenter.microsoft.com/)登入認證。 認證可讓您登入合作夥伴入口網站，然後從夥伴儀表板新增客戶、搜尋客戶，或瀏覽至客戶帳戶。 在 Azure 入口網站中，CSP 可以代表客戶擔任 StorSimple 系統管理員。
                             
## <a name="add-a-customer"></a>新增客戶

新增客戶時會自動建立訂用帳戶。 若要新增客戶 (並自動建立訂用帳戶)，請在合作夥伴入口網站中執行下列步驟。

1. 移至[合作夥伴中心](http://partnercenter.microsoft.com/)，使用 CSP 認證登入。 按一下 [儀表板] 。

     ![合作夥伴中心的儀表板](./media/storsimple-partner-csp-deploy/image1.png)
                              
2. 在左窗格中，按一下 [客戶]。 在右窗格中，按一下 [新增客戶]。 輸入客戶的詳細資料。 按 [下一步︰訂用帳戶] 建立客戶訂用帳戶。

    ![新增客戶](./media/storsimple-partner-csp-deploy/image2.png)

3.  選取 [Microsoft Azure] 優惠。 捲動到頁面底部，按一下 [檢閱]。

    ![檢視訂用帳戶資訊](./media/storsimple-partner-csp-deploy/image3.png)
                              
4. 檢閱資訊，然後按一下 [提交]。

    ![提交訂用帳戶](./media/storsimple-partner-csp-deploy/image4.png)

5. 儲存確認資訊供日後參考。

    ![儲存確認](./media/storsimple-partner-csp-deploy/image5.png)

6. 尋找或瀏覽至您剛才新增的客戶。 按一下 [公司名稱] 向下鑽研詳細資料。

    ![搜尋客戶](./media/storsimple-partner-csp-deploy/image6.png)  

7. 在左窗格中，選取 [服務管理]。 在右窗格中的 [管理服務] 下，按一下 [Microsoft Azure 管理入口網站]，以 Azure 系統管理員身分代表客戶登入。

    ![登入 Azure 入口網站](./media/storsimple-partner-csp-deploy/image9.png)

8. 若要建立 StorSimple 裝置管理員，請按一下 [+ 新增]，然後搜尋或瀏覽至 **StorSimple 虛擬裝置系列**。 如需詳細資訊，請移至[部署 StorSimple 裝置管理員服務](storsimple-virtual-array-manage-service.md)。

    ![建立 StorSimple 裝置管理員服務](./media/storsimple-partner-csp-deploy/image8.png)


## <a name="add-a-subscription"></a>新增訂用帳戶

在某些情況下，您可能已有現有的客戶，您需要新增訂用帳戶。 若要將訂用帳戶新增至現有的客戶，請在合作夥伴入口網站中執行下列步驟。

1. 移至[合作夥伴中心](http://partnercenter.microsoft.com/)，使用 CSP 認證登入。 按一下 [儀表板] 。

     ![合作夥伴中心的儀表板](./media/storsimple-partner-csp-deploy/image1.png)
                              
2. 在左窗格中，按一下 [客戶]。 尋找或瀏覽至客戶您想要新增訂用帳戶的客戶。 按一下 ![展開核取圖示](./media/storsimple-partner-csp-deploy/expand_pane_icon.png) 圖示，展開客戶公司名稱的資料列。 在詳細資料中，按一下 [新增訂用帳戶]。

    ![客戶](./media/storsimple-partner-csp-deploy/image10.png)

3. 在訂用帳戶中，檢查 **Microsoft Azure** 提供的**熱門優惠**，然後按一下 [提交]。 這樣會建立新的訂用帳戶。

    ![新增訂用帳戶](./media/storsimple-partner-csp-deploy/image11.png)

6. 建立新的訂用帳戶之後，在左窗格中按一下 [<-- 客戶]，以返回 [客戶] 頁面。 搜尋您剛建立訂用帳戶的客戶。 按一下 [公司名稱] 向下鑽研詳細資料。

    ![搜尋客戶](./media/storsimple-partner-csp-deploy/image6.png)  

7. 在左窗格中，選取 [服務管理]。 在右窗格中的 [管理服務] 下，按一下 [Microsoft Azure 管理入口網站]，以 Azure 系統管理員身分代表客戶登入。

    ![登入 Azure 入口網站](./media/storsimple-partner-csp-deploy/image9.png)

8. 若要建立 StorSimple 裝置管理員，請按一下 [+ 新增]，然後搜尋或瀏覽至 **StorSimple 虛擬裝置系列**。 如需詳細資訊，請移至[部署 StorSimple 裝置管理員服務](storsimple-virtual-array-manage-service.md)。

    ![建立 StorSimple 裝置管理員服務](./media/storsimple-partner-csp-deploy/image8.png)

## <a name="next-steps"></a>後續步驟

- 關於 StorSimple in CSP，如果您還有其他問題，請移至 [StorSimple in CSP︰常見問題集](storsimple-partner-csp-faq.md)。
- 如果您已準備好部署 StorSimple，請移至[部署 StorSimple in CSP](storsimple-partner-csp-deploy.md)。
