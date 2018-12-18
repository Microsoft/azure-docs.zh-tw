---
title: 針對 StorSimple 8000 系列建立支援票證或案例 | Microsoft Docs
description: 了解如何記錄支援要求和在 StorSimple 8000 系列裝置上啟動支援工作階段。
services: storsimple
documentationcenter: ''
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/09/2018
ms.author: alkohli;
ms.openlocfilehash: e0df86e40d0dc1092ad7ff04f01bbc3e5e3d1c4e
ms.sourcegitcommit: 9292e15fc80cc9df3e62731bafdcb0bb98c256e1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/10/2018
ms.locfileid: "27743038"
---
# <a name="contact-microsoft-support"></a>連絡 Microsoft 支援服務

StorSimple 裝置管理員可讓您在服務摘要刀鋒視窗中**登錄新的支援要求**。 如果您使用 StorSimple 解決方案時遇到任何問題，您可以向技術支援人員提出服務要求。 在與支援工程師進行線上工作階段時，可能也需要在您的 StorSimple 裝置上啟動支援工作階段。 本文將引導您：

* 如何建立支援要求。
* 如何在入口網站上管理支援要求的生命週期。
* 如何在 StorSimple 裝置的 Windows PowerShell 介面中啟動支援工作階段。

建立支援要求之前，請先檢閱 [StorSimple 8000 系列支援 SLA 和資訊](https://msdn.microsoft.com/library/mt433077.aspx) 。

## <a name="create-a-support-request"></a>建立支援要求

視您的[支援方案](https://azure.microsoft.com/support/plans/)而定，您可以直接從 StorSimple 裝置管理員服務摘要刀鋒視窗中，針對 StorSimple 裝置的問題建立支援票證。 執行下列步驟來建立支援要求：

#### <a name="to-create-a-support-request"></a>建立支援要求

1. 移至您的 StorSimple 裝置管理員服務。 在服務摘要刀鋒視窗的設定中，移至 [支援 + 疑難排解] 區段，然後按一下 [新增支援要求]。
     
    ![透過新的入口網站連絡 MS 支援服務](./media/storsimple-8000-contact-microsoft-support/contactsupport1.png)
   
2. 在 [新增支援要求] 刀鋒視窗中，選取 [基本]。 在 [基本] 刀鋒視窗中，執行下列步驟：
   1. 從 [問題類型] 下拉式清單中，選取 [技術]。
   2. 自動會選擇目前的 [訂用帳戶]、[服務] 類型和 [資源] \(StorSimple 裝置管理員服務)。 
   3. 如果您的訂用帳戶有多個相關聯的方案，請從下拉式清單選取**支援方案**。 您必須已付費購買支援方案，才能啟用技術支援。
   4. 按 [下一步] 。

       ![透過新的入口網站連絡 MS 支援服務](./media/storsimple-8000-contact-microsoft-support/contactsupport2.png)

3. 在 [新增支援要求] 刀鋒視窗中，選取 [步驟 2 問題]。 在 [問題] 刀鋒視窗中，執行下列步驟：
    
    1. 選擇 [嚴重性]。
    2. 指定問題是與設備還是 StorSimple 裝置管理員服務有關。
    3. 選擇這個問題的 [類別]，並提供問題的其他 [詳細資料]。
    4. 提供問題開始日期與時間。
    5. 在 [檔案上傳] 中，按一下資料夾圖示，即可瀏覽至您的支援封裝。
    6. 勾選 [共用診斷資訊]。
    7. 按 [下一步] 。

       ![透過新的入口網站連絡 MS 支援服務](./media/storsimple-8000-contact-microsoft-support/contactsupport3.png) 

4. 在 [新增支援要求] 刀鋒視窗中，按一下 [步驟 3 連絡人資訊]。 在 [連絡人資訊] 刀鋒視窗中，執行下列步驟：

    1. 在 [連絡人選項] 中，提供您偏好的連絡方法 (電話或電子郵件) 以及語言。 會根據您的訂用帳戶方案自動選擇回應時間。
    2. 在 [連絡人資訊] 中，提供您的姓名、電子郵件、選用連絡方法、國家/地區。 選取 [儲存連絡人變更以供未來的支援要求使用]  核取方塊。
    3. 按一下頁面底部的 [新增] 。
   
        ![透過新的入口網站連絡 MS 支援服務](./media/storsimple-8000-contact-microsoft-support/contactsupport5.png)   

    Microsoft 支援服務會利用此資訊來連絡您，以取得進一步資訊、進行診斷及解決。
提交您的要求之後，支援工程師會儘速連絡您來處理您的要求。

## <a name="manage-a-support-request"></a>管理支援要求

建立支援票證之後，您便可以從入口網站中管理票證的生命週期。

#### <a name="to-manage-your-support-requests"></a>若要管理支援要求

1. 若要移至說明和支援頁面，請瀏覽至 [瀏覽] > [說明 + 支援]。

    ![管理支援要求](./media/storsimple-8000-contact-microsoft-support/managesupport1.png)

2. [說明 + 支援] 刀鋒視窗中會以表格式清單列出全部的支援要求。

    ![管理支援要求](./media/storsimple-8000-contact-microsoft-support/managesupport2.png)

3. 選取並按一下支援要求。 您可以檢視此要求的狀態與詳細資料。 如果您想要追蹤此要求，請按一下 [+ New message]\(+ 新訊息\)。

    ![管理支援要求](./media/storsimple-8000-contact-microsoft-support/managesupport3.png)

## <a name="start-a-support-session-in-windows-powershell-for-storsimple"></a>在 Windows PowerShell for StorSimple 中啟動支援工作階段

若要對您使用 StorSimple 裝置時可能會遇到的任何問題進行疑難排解，您必須連絡 Microsoft 支援服務小組。 Microsoft 支援服務可能需要使用支援工作階段來登入您的裝置。

執行下列步驟來啟動支援工作階段：

#### <a name="to-start-a-support-session"></a>啟動支援工作階段

1. 從遠端電腦使用序列主控台或透過 telnet 工作階段，直接存取裝置。 如果要這樣做，請依照 [使用 PuTTY 來連接至裝置序列主控台](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console)中的步驟進行。
2. 在開啟的工作階段中，按 **Enter** 鍵來取得命令提示字元。
3. 在序列主控台功能表中，選取選項 1 [使用完整存取權登入] 。
4. 在提示字元中輸入下列密碼：
   
    `Password1`
5. 在出現提示時輸入下列命令：
   
    `Enable-HcsSupportAccess`
6. 您會看到加密的字串。 將此字串複製到 [記事本] 之類的文字編輯器。
7. 儲存這個字串，並透過電子郵件傳送給 Microsoft 支援服務。

> [!IMPORTANT]
> 您可以執行 `Disable-HcsSupportAccess` 來停用支援存取。 在起始工作階段之後 8 小時，StorSimple 裝置也會嘗試停用支援存取。 最好在起始支援工作階段之後變更 StorSimple 裝置認證。


## <a name="next-steps"></a>後續步驟

了解如何[診斷並解決 StorSimple 8000 系列裝置的相關問題](storsimple-8000-troubleshoot-deployment.md)
