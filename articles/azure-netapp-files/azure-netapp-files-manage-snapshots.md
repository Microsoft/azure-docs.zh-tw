---
title: 使用 Azure NetApp Files 管理快照集 | Microsoft Docs
description: 說明如何使用 Azure NetApp Files 建立磁碟區的隨選快照集，或從快照集還原至新的磁碟區。
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
ms.topic: how-to-article
ms.date: 03/28/2018
ms.author: b-juche
ms.openlocfilehash: 48cb88b9815ba723d93c18caf63f33b50eea850c
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39011976"
---
# <a name="manage-snapshots-by-using-azure-netapp-files"></a>使用 Azure NetApp Files 管理快照集
您可以使用 Azure NetApp Files 建立磁碟區的隨選快照集，或從快照集還原至新的磁碟區。

## <a name="create-an-on-demand-snapshot-for-a-volume"></a>建立磁碟區的隨選快照集
您只能以隨選的方式建立快照集。  目前不支援快照集原則。  
1.  在 [管理磁碟區] 刀鋒視窗中按一下 [快照集]，然後按一下 [+ 新增快照集] 以建立磁碟區的隨選快照集。

2.  在 [新增快照集] 視窗中，為您要建立的新快照集提供名稱。   

3. 按一下 [確定]。 


## <a name="restore-a-snapshot-to-a-new-volume"></a>將快照集還原至新磁碟區
目前，您只能將快照集還原至新的磁碟區。 
1. 從 [磁碟區] 刀鋒視窗移至 [管理快照集] 刀鋒視窗，以顯示快照集清單。 
2. 選取要還原的快照集。  
3. 以滑鼠右鍵按一下快照集名稱，然後從功能表選項中選取 [還原至新磁碟區]。  

    ![將快照集還原至新磁碟區](../media/azure-netapp-files/azure-netapp-files-snapshot-restore-to-new-volume.png)

4. 在 [新增磁碟區] 視窗中，提供新磁碟區的資訊：  
    * **名稱**   
        為您要建立的磁碟區指定名稱。  
        
        名稱在資源群組內必須是唯一的。 長度至少必須有 3 個字元。  可以使用任何英數字元。

    * **檔案路徑**     
        指定要用來建立新磁碟區匯出路徑的檔案路徑。 匯出路徑會用來掛接和存取磁碟區。   
        
        掛接目標是 NFS 服務 IP 位址的端點。 這是自動產生的。   
        
        檔案路徑名稱只能包含字母、數字和連字號 ("-")。 長度必須介於 16 到 40 個字元之間。 

    * **配額**  
        指定配置給磁碟區的邏輯儲存體大小。  

        [可用配額] 欄位會顯示所選容量集區中可用來建立新磁碟區的未使用空間量。 新磁碟區的大小不可超過可用配額。

    *   **虛擬網路**  
        指定您要從中存取磁碟區的 Azure 虛擬網路 (Vnet)。 
        
        您指定的 Vnet 必須設定 Azure NetApp Files。 Azure NetApp Files 服務只能從所在位置與磁碟區相同的 Vnet 中存取。  

    ![已還原的新磁碟區](../media/azure-netapp-files/azure-netapp-files-snapshot-new-volume.png) 
    
5. 按一下 [確定]。   
    快照集還原到的新磁碟區會出現在 [磁碟區] 刀鋒視窗中。

