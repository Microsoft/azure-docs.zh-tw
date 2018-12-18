---
title: 存取 Azure 實驗室服務中的教室實驗室 | Microsoft Docs
description: 在本教學課程中，您會在由教師設定的教室實驗室中存取虛擬機器。
services: devtest-lab, lab-services, virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 07/30/2018
ms.author: spelluru
ms.openlocfilehash: dadc90e6a39b9e9689bab0249e6496fdea6f6205
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39450210"
---
# <a name="tutorial-access-a-classroom-lab-in-azure-lab-services"></a>教學課程：存取 Azure 實驗室服務中的教室實驗室
在本教學課程中，您會以學生的身分連線到教室實驗室中的虛擬機器 (VM)。 

在本教學課程中，您會執行下列動作：

> [!div class="checklist"]
> * 使用註冊連結 
> * 連接至虛擬機器

## <a name="use-the-registration-link"></a>使用註冊連結

1. 瀏覽至教師/授課者提供給您的**註冊 URL**。 
2. 使用學校帳戶登入服務，以完成註冊。 
3. 註冊之後，請確認您有看到可存取的實驗室虛擬機器。 
2. 等候虛擬機器就緒，然後**啟動** VM。

    ![啟動 VM](../media/tutorial-connect-vm-in-classroom-lab/start-vm.png)

## <a name="connect-to-the-virtual-machine"></a>連接至虛擬機器

1. 在您要存取的實驗室虛擬機器圖格上，選取 [連線]。 

    ![連接到 VM](../media/tutorial-connect-vm-in-classroom-lab/connect-vm.png)
2. 將 RDP 檔案儲存至硬碟，然後開啟它。 
3. 使用授課者/教師提供給您的**使用者名稱**和**密碼**來登入機器。 

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已可以使用授課者/教師提供給您的註冊連結來存取教室實驗室。

身為實驗室擁有者，您想要檢視已向實驗室註冊的人員，並追蹤 VM 的使用情況。 請前進到下一個教學課程，以了解如何進行此作業：

> [!div class="nextstepaction"]
> [追蹤實驗室的使用情況](tutorial-track-usage.md) 