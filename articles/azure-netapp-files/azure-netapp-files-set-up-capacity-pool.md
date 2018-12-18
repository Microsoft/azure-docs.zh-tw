---
title: 設定 Azure NetApp Files 的容量集區 | Microsoft Docs
description: 說明如何設定容量集區，以便您可以在其中建立磁碟區。
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 03/28/2018
ms.author: b-juche
ms.openlocfilehash: 0e9203f5b4e2a9043e242b804c82017cf6fc3ee1
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39010798"
---
# <a name="set-up-a-capacity-pool"></a>設定容量集區
設定容量集區可讓您在其中建立磁碟區。  

## <a name="before-you-begin"></a>開始之前 
您必須已經建立 NetApp 帳戶。   

[建立 NetApp 帳戶](azure-netapp-files-create-netapp-account.md)

## <a name="steps"></a>步驟 

1. 請移至 NetApp 帳戶的 [管理] 刀鋒視窗，然後從導覽窗格中，選取 [容量集區]。

2. 按一下 [+ 新增集區] 來建立新的容量集區。   
    [新增容量集區] 視窗隨即出現。

3. 請提供新容量集區的下列資訊：  
  * **名稱**  
    指定容量集區的名稱。  
    每個 NetApp 帳戶的容量集區名稱必須是唯一的。

  * **服務等級**   
    此欄位會顯示容量集區的目標效能。  
    目前只有進階服務等級可用。 

  *  **大小**     
      指定您購買的容量集區大小。        
      最小容量集區大小是 4 TB。 您可以建立大小是 4 TB 倍數的集區。   
      
      ![新增容量集區](../media/azure-netapp-files/azure-netapp-files-new-capacity-pool.png)

4. 按一下 [確定]。

## <a name="next-steps"></a>後續步驟 

1. [建立適用於 Azure NetApp Files 的磁碟區](azure-netapp-files-create-volumes.md)
2. [設定磁碟區的匯出原則 (選用)](azure-netapp-files-configure-export-policy.md)

