---
title: 將內部支援陳述式新增到 Azure DevTest Labs 中的實驗室 | Microsoft Docs
description: 了解如何將內部支援陳述式張貼到 Azure DevTest Labs 中的實驗室
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: c9900333-6c5e-4d7d-b0f4-889015e9550c
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 2d12ca26fb2aa5abddcf44b2e634b2f08b1fb01b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38696245"
---
# <a name="add-an-internal-support-statement-to-a-lab-in-azure-devtest-labs"></a>將內部支援陳述式新增到 Azure DevTest Labs 中的實驗室

Azure DevTest Labs 可讓您使用為使用者提供實驗室相關支援資訊的內部支援陳述式，自訂您的實驗室。 例如，您可以提供連絡資訊，讓使用者知道在需要協助以進行疑難排解，或存取實驗室中的資源時，如何連絡內部支援。 您也可以提供指向內部網站或常見問題集的連結，您的小組可以在連絡支援人員之前，先存取這些資源。

內部支援陳述式旨在讓您張貼通常不太常變更的實驗室資訊。 若要通知使用者本質上更臨時的實驗室資訊 (例如實驗室原則最近的更新)，請參閱[在實驗室中張貼公告](devtest-lab-announcements.md)。

在支援陳述式不再適用之後，您就可以輕鬆地加以停用或編輯。

## <a name="steps-to-add-a-support-statement-to-an-existing-lab"></a>將支援陳述式新增到現有實驗室的步驟

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 視需要選取 [所有服務]，然後從清單中選取 [DevTest Labs]。 (您的實驗室可能已顯示在 [所有資源] 下方的儀表板上)。
1. 從實驗室清單中，選取您想要在其中新增支援陳述式的實驗室。  
1. 在實驗室的 [概觀] 區域中選取 [組態和原則]。  

    ![組態和原則按鈕](./media/devtest-lab-internal-support-message/devtestlab-config-and-policies.png)

1. 在左側的 [設定] 之下，選取 [內部支援]。

    ![內部支援按鈕](./media/devtest-lab-internal-support-message/devtestlab-internal-support.png)

1. 若要在此實驗室中為使用者建立內部支援訊息，請將 [啟用] 設定為 [是]。

1. 在 [支援訊息] 欄位中，輸入您想要向實驗室使用者呈現的內部支援陳述式。 支援訊息接受 Markdown。 當您輸入訊息文字時，可以檢視畫面底部的 [預覽] 區域，以查看訊息向使用者呈現的方式。

    ![用於建立訊息的內部支援畫面。](./media/devtest-lab-internal-support-message/devtestlab-add-support-statement.png)


1. 一旦準備好張貼支援陳述式之後，請選取 [儲存]。

當您不想要再對實驗室使用者顯示此支援訊息時，請返回 [內部支援] 頁面，然後將 [啟用] 設定為 [否]。

## <a name="steps-for-users-to-view-the-support-message"></a>讓使用者檢視支援訊息的步驟

1. 從 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)選取一個實驗室。

1. 在實驗室的 [概觀] 區域上，選取 [內部支援]。  

    ![內部支援按鈕](./media/devtest-lab-internal-support-message/devtestlab-internal-support.png)


1. 如果已張貼支援訊息，使用者就可以從內部支援窗格檢視該訊息。

    ![顯示已張貼之支援訊息的內部支援窗格](./media/devtest-lab-internal-support-message/devtestlab-view-suport-statement.png)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
* 內部支援陳述式通常用來提供不會經常變更的支援資訊。 您也可以了解如何[將公告張貼到實驗室](devtest-lab-announcements.md)以通知使用者臨時變更或實驗室的更新。
* [設定原則和排程](devtest-lab-set-lab-policy.md)提供的資訊是關於如何使用自訂的原則，在訂用帳戶內套用其他限制和慣例。