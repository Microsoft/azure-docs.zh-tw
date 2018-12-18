---
title: 使用 Marketplace 工具組來建立和發佈 Marketplace 項目 | Microsoft Docs
description: 了解如何以發佈工具組快速建立 Marketplace 項目
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 9/26/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: 518384776f777c3ea7cbd636dd2a48d919952f28
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409111"
---
#  <a name="add-marketplace-items-using-publishing-tool"></a>使用發佈工具新增 Marketplace 項目

將您的內容新增到 [Azure Stack Marketplace](azure-stack-marketplace.md)，可讓您的解決方案供您和您的租用戶用於部署。 Marketplace 工具組會根據您的 IaaS Azure Resource Manager 範本或 VM 擴充功能，建立 Azure Marketplace 套件 (.azpkg) 檔案。 您也可以使用 Marketplace 工具組來發佈 .azpkg 檔案 (可使用此工具或使用[手動](azure-stack-create-and-publish-marketplace-item.md)步驟建立)。 本主題說明如何下載工具、根據 VM 範本建立 Marketplace 項目，然後將該項目發佈到 Azure Stack Marketplace。     

## <a name="prerequisites"></a>必要條件

 - 您必須在 Azure Stack 主機上執行此工具組，或在執行工具所在的電腦上具有 [VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn) 連線能力。

 - 下載 [Azure Stack 快速入門範本](https://github.com/Azure/AzureStack-QuickStart-Templates/archive/master.zip)並解壓縮。

 - 下載 [Azure 資源庫封裝工具](http://aka.ms/azurestackmarketplaceitem) (AzureGalleryPackage.exe)。 

 - 要發佈到 Marketplace 需要有圖示和縮圖檔案。 您可以使用自己的檔案，或是將此範例的[範例](azure-stack-marketplace-publisher.md#support-files)檔案儲存到本機上。

## <a name="download-the-tool"></a>下載工具

您可從 [Azure Stack 工具存放庫](azure-stack-powershell-download.md)下載 Marketplace 工具組。

##  <a name="create-marketplace-items"></a>建立 Marketplace 項目

在本節中，您會使用 Marketplace 工具組建立 .azpkg 格式的 Marketplace 項目套件。  

### <a name="provide-marketplace-information-with-wizard"></a>以精靈提供 Marketplace 資訊

1. 從 PowerShell 工作階段中執行 Marketplace 工具組：
   ```PowerShell
   .\MarketplaceToolkit.ps1
   ```

2. 按一下 [解決方案] 索引標籤。此畫面會接受您的 Marketplace 項目相關資訊。 請輸入您想要顯示在 Marketplace 中的項目相關資訊。 您也可以指定[參數檔案](azure-stack-marketplace-publisher.md#use-a-parameters-file)來預先填入表單。  
    
    ![Marketplace 工具組第一個畫面的螢幕擷取畫面](./media/azure-stack-marketplace-publisher/image7.png)
3. 按一下 [瀏覽] 以選取每個圖示和螢幕擷取畫面欄位的影像檔。 您可以使用自己的圖示，或[支援檔案](azure-stack-marketplace-publisher.md#support-files)一節中的範例圖示。
4. 一旦所有欄位都已填入，請選取「預覽解決方案」來預覽 Marketplace 內的解決方案。 您可以修改並編輯文字、影像和螢幕擷取畫面後，再按一下 [下一步]。  

### <a name="import-template-and-create-package"></a>匯入範本並建立套件

在本節中，您會匯入範本並使用解決方案的輸入。

1.  按一下 [瀏覽] 並從所下載範本的 101-Simple-Windows-VM 資料夾中選取 *azuredeploy.json*。

    ![Marketplace 工具組第二個畫面的螢幕擷取畫面](./media/azure-stack-marketplace-publisher/image8.png)
2.  「部署精靈」中會填入 [基本] 步驟，以及範本中所指定每個參數的輸入項目。 您可以新增其他步驟，並在步驟之間移動輸入項。 例如，您可能會想要為您的解決方案新增「前端設定」和「後端設定」步驟。
3.  請指定 AzureGalleryPackager.exe 的路徑。  
4.  按一下頁面底部的 [新增] 。 Marketplace 工具組會將您的解決方案封裝到 .azpkg 檔案中。 完成後，精靈會顯示您的解決方案檔案路徑，並讓您選擇繼續將您的套件發佈到 Azure Stack。

## <a name="publish-marketplace-items"></a>發佈 Marketplace 項目

在本節中，您會將您的 Marketplace 項目發佈到 Azure Stack Marketplace。

![Marketplace 工具組第一個畫面的螢幕擷取畫面](./media/azure-stack-marketplace-publisher/image9.png)

1.  精靈需要下列資訊來發佈您的解決方案：
    
    |欄位|說明|
    |-----|-----|
    | 服務管理員名稱 | 服務管理員帳戶。  範例： ServiceAdmin@mydomain.onmicrosoft.com |
    | 密碼 | 服務管理員帳戶的密碼。 |
    | API 端點 | Azure Stack Azure Resource Manager 端點。 範例：management.local.azurestack.external |
2.  按一下 [發佈]，發佈記錄隨即顯示。
3.  您現在已可透過 Azure Stack 入口網站部署您發佈的項目。

## <a name="use-a-parameters-file"></a>使用參數檔案

您也可以使用參數檔案來完成 Marketplace 項目資訊。  

Marketplace 工具組包含 *solution.parameters.ps1*，可用來建立您自己的參數檔案。

## <a name="support-files"></a>支援檔案

| 說明 | 範例 |
| ----- | ----- |
| 40 x 40.png 圖示 | ![](./media/azure-stack-marketplace-publisher/image1.png) |
| 90x90 .png 圖示 | ![](./media/azure-stack-marketplace-publisher/image2.png) |
| 115x115 .png 圖示 | ![](./media/azure-stack-marketplace-publisher/image3.png) |
| 255x115 .png 圖示 | ![](./media/azure-stack-marketplace-publisher/image4.png) |
| 533x324 .png 縮圖 | ![](./media/azure-stack-marketplace-publisher/image5.png) |

## <a name="next-steps"></a>後續步驟

[下載 Marketplace 項目](azure-stack-download-azure-marketplace-item.md)  
[建立及發佈 Marketplace 項目](azure-stack-create-and-publish-marketplace-item.md)