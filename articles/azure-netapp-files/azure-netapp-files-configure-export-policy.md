---
title: 設定 Azure NetApp Files 磁碟區的匯出原則 | Microsoft Docs
description: 說明如何設定匯出原則以控制 Azure NetApp Files 磁碟區的存取權
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
ms.openlocfilehash: 08de7e2ca8cd993a0931f5b16cb9d6c9a04e53dc
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39010951"
---
# <a name="configure-export-policy-for-a-volume-optional"></a>設定磁碟區的匯出原則 (選用)

您可以選擇性地設定匯出原則，以控制 Azure NetApp Files 磁碟區的存取權。 

## <a name="steps"></a>步驟 

1.  從 [管理磁碟區] 刀鋒視窗按一下 [建立匯出原則] 刀鋒視窗。 

2.  指定下列欄位的資訊以建立匯出原則規則：   
    *  **索引**   
        指定規則的索引編號。  
        匯出原則可以包含最多五項規則。 系統會會根據規則在索引編號清單中的順序評估規則。 具有較低索引編號的規則會先接受評估。 例如，具有索引編號 1 的規則，會在具有索引編號 2 的規則之前接受評估。 

    * **允許的用戶端**   
        以下列其中一個格式指定值：  
        * IPv4 位址，例如 `10.1.12.24` 
        * IPv4 位址，具有以位元數表示的子網路遮罩，例如 `10.1.12.10/4`

    * **Access**  
        請選取下列其中一個存取類型：  
        * 無存取權 
        * 讀取和寫入
        * 唯讀

    * **通訊協定**   
        指定要用於匯出原則的通訊協定。   
        目前 Azure NetApp Files 僅支援 NFSv3。

    ![匯出原則](../media/azure-netapp-files/azure-netapp-files-export-policy.png) 


## <a name="next-steps"></a>後續步驟 
* [管理磁碟區](azure-netapp-files-manage-volumes.md)
* [裝載或取消裝載虛擬機器磁碟區](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)
* [管理快照集](azure-netapp-files-manage-snapshots.md)
