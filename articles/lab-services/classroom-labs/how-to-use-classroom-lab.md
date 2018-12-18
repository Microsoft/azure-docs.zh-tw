---
title: 如何存取 Azure 實驗室服務中的教室實驗室 | Microsoft Docs
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
ms.date: 05/17/2018
ms.author: spelluru
ms.openlocfilehash: 85da6989740446c980bcb9f2c89a6c31afe488fb
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34651324"
---
# <a name="how-to-access-a-classroom-lab-in-azure-lab-services"></a>如何存取 Azure 實驗室服務中的教室實驗室
本文將說明如何存取教室實驗室、連線到實驗室中的 VM，以及停止 VM。 

## <a name="view-all-the-classroom-labs"></a>檢視所有教室實驗室

1. 瀏覽至教師/授課者提供給您的**註冊 URL**。 
2. 使用學校帳戶登入服務，以完成註冊。 
3. 註冊之後，請確認您有看到可存取的實驗室虛擬機器。 

    ![檢視所有連結](../media/how-to-use-classroom-lab/all-labs.png)

## <a name="connect-to-the-virtual-machine-in-a-classroom-lab"></a>連線到教室實驗室中的虛擬機器

1. 在您要存取的實驗室虛擬機器圖格上，選取 [連線]。

    ![檢視所有連結](../media/how-to-use-classroom-lab/connect-button.png)
2. 讓虛擬機器就緒需要幾分鐘的時間。

    ![正在準備虛擬機器](../media/how-to-use-classroom-lab/getting-virtual-machine-ready.png)
3. 將 RDP 檔案儲存至硬碟，然後開啟它。 
    
    ![儲存 RDP 檔案](../media/how-to-use-classroom-lab/save-rdp-file.png)
4. 使用授課者/教師提供給您的**使用者名稱**和**密碼**來登入機器。 

## <a name="stop-the-virtual-machine-in-a-classroom-lab"></a>停止教室實驗室中的虛擬機器

在代表教室實驗室的圖格上選取 [停止]。 當 VM 停止時，在圖格上的 [啟動] 按鈕會啟用。 

## <a name="next-steps"></a>後續步驟
使用 Azure 實驗室服務以設定實驗室的入門：

- [設定教室實驗室](how-to-manage-classroom-labs.md)
- [設定實驗室](../tutorial-create-custom-lab.md)

 