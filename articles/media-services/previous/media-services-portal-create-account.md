---
title: 使用 Azure 入口網站建立 Azure 媒體服務帳戶 | Microsoft Docs
description: 本教學課程逐步引導您完成使用 Azure 入口網站建立 Azure 媒體服務帳戶的步驟。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: c551e158-aad6-47b4-931e-b46260b3ee4c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/01/2018
ms.author: juliako
ms.openlocfilehash: da190bf2418f1cfb8ea952b69d3bf1d76258da5f
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37342322"
---
# <a name="create-an-azure-media-services-account-using-the-azure-portal"></a>使用 Azure 入口網站建立 Azure 媒體服務帳戶
> [!div class="op_single_selector"]
> * [入口網站](media-services-portal-create-account.md)
> * [REST](https://docs.microsoft.com/rest/api/media/mediaservice)
> 
> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
> 
> 

Azure 入口網站提供一種方法來快速建立 Azure 媒體服務 (AMS) 帳戶。 您可以使用自己的帳戶，來存取讓您在 Azure 中儲存、加密、編碼、管理和串流播放媒體內容的媒體服務。 當您建立媒體服務帳戶時，您也會在與媒體服務帳戶相同的地理區域中建立相關聯的儲存體帳戶 (或使用現有儲存體帳戶)。 

您的主要儲存體帳戶可以是一般用途 v1 或一般用途 v2。 目前，Azure 入口網站只允許挑選 v1，但是您可以在使用 API 或 Powershell 建立帳戶時新增 v2。 如需儲存體類型的詳細資訊，請參閱[關於 Azure 儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account)。

本文說明一些常見的概念，並示範如何使用 Azure 入口網站建立媒體服務帳戶。

> [!NOTE]
> 如需不同區域中 Azure 媒體服務功能可用性的資訊，請參閱[跨資料中心的 AMS 功能可用性](scenarios-and-availability.md#availability)。

## <a name="concepts"></a>概念
存取媒體服務時需要有兩個相關聯的帳戶：

* 媒體服務帳戶。 您的帳戶可讓您存取 Azure 中提供的一組雲端式媒體服務資源。 媒體服務帳戶並不會儲存實際媒體內容。 而是在您的帳戶中儲存媒體內容和媒體處理工作的中繼資料。 當您建立帳戶時，您會選取一個可用的媒體服務區域。 所選取的區域會是儲存您帳戶之中繼資料記錄的資料中心。
  
* 一個 Azure 儲存體帳戶。 儲存體帳戶必須與媒體服務帳戶位於相同的地理區域中。 建立媒體服務帳戶時，可以選擇相同區域中的現有儲存體帳戶，也可以在相同區域中建立新的儲存體帳戶。 如果您刪除媒體服務帳戶，並不會刪除相關儲存體帳戶中的 Blob。

## <a name="create-an-ams-account"></a>建立 AMS 帳戶
本節中的步驟示範如何建立 AMS 帳戶。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下 [+新增] > [Web + 行動] > [媒體服務]。
   
    ![建立媒體服務](./media/media-services-create-account/media-services-new1.png)
3. 在 [建立媒體服務帳戶]  中輸入必要的值。
   
    ![建立媒體服務](./media/media-services-create-account/media-services-new3.png)
   
   1. 在 [帳戶名稱] 中，輸入新 AMS 帳戶的名稱。 媒體服務帳戶名稱為全部小寫且不含空格的字母或數字，且長度是 3 到 24 個字元。
   2. 在訂用帳戶中，從您可存取的不同 Azure 訂用帳戶中進行選取。
   3. 在 [資源群組] 中，選取新的或現有資源。  資源群組是共用生命週期、權限及原則的資源集合。 [在此](../../azure-resource-manager/resource-group-overview.md#resource-groups)深入了解。
   4. 在 [位置] 中，選取您將用來為媒體服務帳戶儲存媒體和中繼資料記錄的地理區域。 此區域將用於處理和串流媒體。 只有可用的媒體服務區域才會出現在下拉式清單方塊中。 
   5. 在 [儲存體帳戶] 中，選取儲存體帳戶以從媒體服務帳戶提供媒體內容的 Blob 儲存體。 您可以選取與媒體服務帳戶相同地理區域中的現有儲存體帳戶，也可以建立儲存體帳戶。 新的儲存體帳戶會建立於相同的區域中。 儲存體帳戶名稱的規則會與媒體服務帳戶相同。
      
       在 [這裡](../../storage/common/storage-introduction.md)深入了解儲存體。
   6. 選取 **[釘選到儀表板]**  以查看帳戶部署的進度。
4. 按一下表單底部的 [建立]  。
   
    成功建立帳戶後，隨即載入概觀頁面。 在串流端點資料表中，此帳戶將具有 [已停止] 狀態的預設串流端點。 

    >[!NOTE]
    >建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。 
   
## <a name="to-manage-your-ams-account"></a>管理 AMS 帳戶

若要管理 AMS 帳戶 (例如，以程式設計方式連線至 AMS API、上傳影片、編碼資產、設定內容保護、監視作業進度)，請選取入口網站左側的 [設定]。 從 [設定] 中，導覽至其中一個可用的刀鋒視窗 (例如：[API 存取]、[資產]、[作業]、[內容保護])。


## <a name="next-steps"></a>後續步驟

您現在可以將檔案上傳到 AMS 帳戶。 如需詳細資訊，請參閱 [上傳檔案](media-services-portal-upload-files.md)。

如果您要以程式設計方式存取 AMS API，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

