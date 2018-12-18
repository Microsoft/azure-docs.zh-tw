---
title: 建立 NetApp 帳戶以存取 Azure NetApp Files | Microsoft Docs
description: 說明如何存取 Azure NetApp Files 及建立 NetApp 帳戶，以便設定容量集區及建立磁碟區。
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
ms.openlocfilehash: ad8cc550ce69e4dc4c19a569718fa873a65b3620
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39010339"
---
# <a name="create-a-netapp-account"></a>建立 NetApp 帳戶
建立 NetApp 帳戶可讓您設定容量集區，並接著建立磁碟區。 您可以使用 [Azure NetApp Files] 刀鋒視窗來建立新的 NetApp 帳戶。

## <a name="before-you-begin"></a>開始之前
您必須已加入白名單，才能存取 Microsoft.NetApp Azure 資源提供者，並且設定以使用 Azure NetApp Files 服務。  

[Azure NetApp Files 公開預覽註冊頁面](https://aka.ms/nfspublicpreview)。 

## <a name="steps"></a>步驟 

1. 從您的預覽邀請找到預覽 Azure 入口網站 URL，然後登入入口網站。 
2.  使用下列其中一個方法存取 [Azure NetApp Files] 刀鋒視窗：  
  * 在 Azure 入口網站搜尋方塊中搜尋 **Azure NetApp Files**。  
  * 按一下導覽中的 [所有服務]，然後篩選為 Azure NetApp Files。  

  您可以藉由按一下 [Azure NetApp Files] 刀鋒視窗旁邊的星號，將其加入「我的最愛」。 

3. 按一下 [+ 新增] 以建立新的 NetApp 帳戶。  
  [新增 NetApp 帳戶] 視窗隨即出現。  

4. 請提供 NetApp 帳戶的下列資訊： 
  * **帳戶名稱**  
    指定訂用帳戶的唯一名稱。
  *  **訂用帳戶**  
    從現有訂用帳戶選取訂用帳戶。
  * **資源群組**   
    使用現有的資源群組或建立新群組。
  * **位置**  
    選取您想要帳戶及其子資源位在哪個區域。  
    目前，Azure NetApp Files 服務僅在美國東部區域受到支援。  

    ![新增 NetApp 帳戶](../media/azure-netapp-files/azure-netapp-files-new-netapp-account.png)


5. 按一下頁面底部的 [新增] 。     
  您剛建立的 NetApp 帳戶現在會出現在 [Azure NetApp Files] 刀鋒視窗中。 

## <a name="next-steps"></a>後續步驟  

1. [設定容量集區](azure-netapp-files-set-up-capacity-pool.md)
2. [建立適用於 Azure NetApp Files 的磁碟區](azure-netapp-files-create-volumes.md)
3. [設定磁碟區的匯出原則 (選用)](azure-netapp-files-configure-export-policy.md)
