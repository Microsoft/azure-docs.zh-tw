---
title: Microsoft Azure 資料箱磁碟的快速入門 | Microsoft Docs
description: 使用此快速入門，在 Azure 入口網站中快速部署您的 Azure 資料箱磁碟
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: quickstart
ms.date: 09/07/2018
ms.author: alkohli
Customer intent: As an IT admin, I need to quickly deploy Data Box Disk so as to import data into Azure.
ms.openlocfilehash: ca0fb177c8b6a16cf624fd4533c9c9040e6d9dee
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49365229"
---
# <a name="quickstart-deploy-azure-data-box-disk-using-the-azure-portal-preview"></a>快速入門：使用 Azure 入口網站部署 Azure 資料箱磁碟 (預覽)

本快速入門說明如何使用 Azure 入口網站來部署 Azure 資料箱磁碟。 這些步驟包含如何快速建立訂單、接收磁碟、打開包裝、連線及將資料複製到磁碟，以便上傳至 Azure。 

如需部署和追蹤的詳細逐步指示，請移至[教學課程：訂購 Azure 資料箱磁碟](data-box-disk-deploy-ordered.md)。 

如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

> [!IMPORTANT]
> 資料箱磁碟處於預覽狀態。 部署這個解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="prerequisites"></a>必要條件

開始之前：

- 請確定您已啟用 Azure 資料箱服務的訂用帳戶。 若要啟用這項服務的訂用帳戶，請[註冊服務](http://aka.ms/azuredataboxfromdiskdocs)。

## <a name="sign-in-to-azure"></a>登入 Azure

在 [http://aka.ms/azuredataboxfromdiskdocs](http://aka.ms/azuredataboxfromdiskdocs) 登入 Azure 入口網站。

## <a name="order"></a>順序

這個步驟需要大約 5 分鐘。

1. 在 Azure 入口網站中建立新的 Azure 資料箱資源。 
2. 選取針對此服務啟用的訂用帳戶，然後選擇 [匯入] 作為轉移類型。 提供資料所在的 [來源國家/地區]，以及資料傳輸的 [Azure 目的地區域]。
3. 選取 [資料箱磁碟]。 解決方案的容量上限為 35 TB，您可以針對較大的資料大小建立多個磁碟訂單。  
4. 輸入訂單詳細資料和出貨資訊。 如果您的區域可使用服務，請提供通知電子郵件地址、檢閱摘要，然後建立訂單。 

一旦建立訂單後，磁碟就準備出貨。 

## <a name="unpack"></a>打開包裝

這個步驟需要大約 5 分鐘。

資料箱磁碟會以 UPS 快遞包裹寄送。 開啟包裹，並檢查包裹中包含：

- 1 到 5 個發泡包裝的 USB 磁碟。
- 每個磁碟都有連接纜線。 
- 退貨用的出貨標籤。

## <a name="connect-and-unlock"></a>連線並解除鎖定

這個步驟需要大約 5 分鐘。

1. 使用內含的纜線將磁碟連接到執行支援版本的 Windows/Linux 電腦。 如需有關支援作業系統版本的詳細資訊，請移至 [Azure 資料箱磁碟系統需求](data-box-disk-system-requirements.md)。 
2. 若要將磁碟解除鎖定：

    1. 在 Azure 入口網站中，前往 [一般] > [裝置詳細資料] 並取得通行金鑰。
    2. 在用來將資料複製到磁碟上的電腦上，下載及擷取作業系統特定的資料箱碟解除鎖定工具。 
    3. 執行資料箱磁碟解除鎖定工具，並提供通行金鑰。 若要執行磁碟重新插入，請再次執行解除鎖定工具，並提供通行金鑰。 **請勿使用 BitLocker 對話方塊或 BitLocker 金鑰來解除鎖定磁碟。** 如需如何將磁碟解除鎖定的詳細資訊，請移至[解除鎖定 Windows 用戶端上的磁碟]()或[解除鎖定 Linux 用戶端上的磁碟]()。
    4. 指派給磁碟的磁碟機代號會由工具顯示。 請記下磁碟機代號。 這會用於後續的步驟。

## <a name="copy-data-and-verify"></a>複製資料並加以確認

完成這項作業的時間取決於您的資料大小。 

1. 磁碟機包含 PageBlob、BlockBlob、AzureImportExport 資料夾。 拖放需要以區塊 Blob 形式匯入的資料，並複製到 BlockBlob 資料夾。 同樣地，將 VHD/VHDX 等資料拖放到 PageBlob 資料夾。

    系統會在 Azure 儲存體帳戶中，為 BlockBlob 和 PageBlob 資料夾下的每個子資料夾建立容器。 BlockBlob 和 PageBlob 資料夾下的所有檔案，都會複製到 Azure 儲存體帳戶下的預設容器 `$root`。

    > [!NOTE] 
    > - 所有容器和 Blob 都應符合 [Azure 命名慣例](data-box-disk-limits.md#azure-block-blob-and-page-blob-naming-conventions)。 如果未遵循這些規則，則將資料上傳至 Azure 會失敗。
    > - 請確定檔案的區塊 Blob 不超過約 4.75 TiB，而分頁 Blob 不超過約 8 TiB。

2. (選擇性) 複製完成之後，建議您執行 AzureImportExport 資料夾中提供的 `DataBoxDiskValidation.cmd`，以產生驗證的總和檢查碼。 視資料大小而定，此步驟可能需要一段時間。 
3. 拔除磁碟機。 


## <a name="ship-to-azure"></a>寄送到 Azure

此步驟約需要 5-7 分鐘來完成。

1. 將所有磁碟一起放在原始套件中。 使用所包含的出貨標籤。 如果標籤受損或遺失，請從入口網站下載。 移至 [概觀]，然後從命令列中按 [下載出貨標籤]。
2. 將已密封的包裹交給運送公司。  

資料箱磁碟服務會傳送電子郵件通知，並且在 Azure 入口網站上更新訂單狀態。


## <a name="verify-your-data"></a>驗證資料

完成這項作業的時間取決於您的資料大小。

1. 當資料箱磁碟連線到 Azure 資料中心網路時，上傳至 Azure 的資料會自動啟動。 
2. Azure 資料箱服務會通知您，已透過 Azure 入口網站完成資料複製。 
    
    1. 檢查所有失敗的錯誤記錄，並採取適當的動作。
    2. 請先確認您的資料位於儲存體帳戶中，再從來源予以刪除。

## <a name="clean-up-resources"></a>清除資源

完成此步驟需要 2-3 分鐘。

若要清除，您可以取消資料箱訂單，然後再加以刪除。

- 訂單處理之前，您可以取消 Azure 入口網站中的資料箱訂單。 一旦處理訂單後，就無法取消訂單。 訂單會進行直到達到已完成的階段為止。 

    若要取消訂單，請前往 [概觀]，然後從命令列按一下 [取消]。  

- 一旦 Azure 入口網站中的狀態顯示為 [已完成] 或是 [已取消] 後，您就可以刪除訂單。 

    若要刪除訂單，請前往 [概觀]，然後從命令列按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署 Azure 資料箱磁碟，以協助將您的資料匯入 Azure。 若要深入了解 Azure 資料箱磁碟管理，請進入下列教學課程： 

> [!div class="nextstepaction"]
> [使用 Azure 入口網站來管理資料箱磁碟](data-box-portal-ui-admin.md)


