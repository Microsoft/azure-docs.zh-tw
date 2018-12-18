---
title: 在 Azure Stack 中提供虛擬機器擴展集 | Microsoft Docs
description: 了解雲端操作員如何將虛擬機器擴展集新增至 Azure Stack Marketplace
services: azure-stack
author: brenduns
manager: femila
editor: ''
ms.service: azure-stack
ms.topic: article
ms.date: 06/05/2018
ms.author: brenduns
ms.reviewer: kivenkat
ms.openlocfilehash: 4e77e187d969af7ea2a12754b18d4a218daceed6
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39411901"
---
# <a name="make-virtual-machine-scale-sets-available-in-azure-stack"></a>在 Azure Stack 中提供虛擬機器擴展集

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

虛擬機器擴展集是 Azure Stack 計算資源。 您可以使用它們來部署和管理一組相同的虛擬機器。 由於所有虛擬機器具有相同的設定，擴展集不需要預先佈建虛擬機器。 您可以更輕鬆地針對大量計算、巨量資料和容器化工作負載，建置大規模服務。

本文引導您完成讓擴展集可在 Azure Stack Marketplace 中使用的程序。 完成此程序之後，您的使用者可以將虛擬機器擴展集新增至其訂用帳戶。

Azure Stack 上的虛擬機器擴展集就像是 Azure 上的虛擬機器擴展集。 如需詳細資訊，請參閱下列視訊：
* [Mark Russinovich 講述 Azure 擴展集](https://channel9.msdn.com/Blogs/Regular-IT-Guy/Mark-Russinovich-Talks-Azure-Scale-Sets/)
* [Guy Bowerman 與虛擬機器擴展集](https://channel9.msdn.com/Shows/Cloud+Cover/Episode-191-Virtual-Machine-Scale-Sets-with-Guy-Bowerman)

在 Azure Stack 上，虛擬機器擴展集不支援自動擴展。 您可以使用 Resource Manager 範本、CLI 或 PowerShell 將更多執行個體新增到擴展集。

## <a name="prerequisites"></a>必要條件

- **Marketplace**  
    向全域 Azure 註冊 Azure Stack 以在 Marketplace 中啟用項目的可用性。 請遵循[向 Azure 註冊 Azure Stack](azure-stack-registration.md) 中的指示。
- **作業系統映像**  
    如果您尚未在 Azure Stack Marketplace 中新增作業系統映像，請參閱[從 Azure 新增 Azure Stack 市集項目](asdk/asdk-marketplace-item.md)。

## <a name="add-the-virtual-machine-scale-set"></a>新增虛擬機器擴展集

1. 開啟 Azure Stack Marketplace 並連線至 Azure。 選取 [Marketplace 管理]> [+ 從 Azure 新增]。

    ![Marketplace 管理](media/azure-stack-compute-add-scalesets/image01.png)

2. 新增並下載虛擬機器擴展集市集項目。

    ![虛擬機器擴展集](media/azure-stack-compute-add-scalesets/image02.png)

## <a name="update-images-in-a-virtual-machine-scale-set"></a>更新虛擬機器擴展集中的映像

建立虛擬機器擴展集之後，使用者可以更新擴展集中的映像，而不必重新建立擴展集。 更新映像的程序取決於下列案例：

1. 虛擬機器擴展集部署範本會**指定最新的** version：  

   當 version 在擴展集範本的 imageReference 區段中設定為 **latest** 時，擴展集的相應放大作業會針對擴展集執行個體使用映像可用的最新版本。 相應放大完成之後，您可以刪除較舊的虛擬機器擴展集執行個體。  (publisher、offer 和 sku 的值維持不變)。 

   以下是指定 latest 的範例：  

    ```Json  
    "imageReference": {
        "publisher": "[parameters('osImagePublisher')]",
        "offer": "[parameters('osImageOffer')]",
        "sku": "[parameters('osImageSku')]",
        "version": "latest"
        }
    ```

   您必須先下載新映像，相應放大才可使用這個新映像：  

   - 當 Marketplace 上的映像版本比擴展集中的映像還要新時：下載新映像以取代舊映像。 取代映像之後，使用者可以繼續相應放大。 

   - 當 Marketplace 上的映像版本與擴展集中的映像相同時：刪除擴展集中使用的映像，然後下載新映像。 在移除原始映像與下載新映像之間的這段時間內，您無法相應放大。 
      
     您必須執行此程序，才能重新整合使用疏鬆檔案格式 (在版本 1803 中引進) 的映像。 
 

2. 虛擬機器擴展集部署範本**不會指定最新的** version，而會改為指定一個版本號碼：  

    如果您下載較新版本的映像 (它會變更可用版本)，則無法相應放大擴展集。 這是因為根據設計，擴展集範本中指定的映像版本必須可以使用。  

如需詳細資訊，請參閱[作業系統磁碟和映像](.\user\azure-stack-compute-overview.md#operating-system-disks-and-images)。  


## <a name="remove-a-virtual-machine-scale-set"></a>移除虛擬機器擴展集

若要移除虛擬機器擴充集資源庫項目，請執行下列 PowerShell 命令：

```PowerShell  
    Remove-AzsGalleryItem
````

> [!NOTE]
> 資源庫項目可能不會立即移除。 您可能需要重新整理入口網站數次，項目才會顯示已從 Marketplace 中移除。

## <a name="next-steps"></a>後續步驟
[Azure Stack 的常見問題集](azure-stack-faq.md)