---
title: 從 Azure StorSimple 將檔案上傳至 Azure 媒體服務帳戶 | Microsoft Docs
description: 本文提供 Azure StorSimple Data Manager 的簡短概述。 本文也會連結至教學課程，說明如何從 StorSimple 擷取資料，並將它當作資產上傳至 Azure 媒體服務帳戶。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 1dd09328-262b-43ef-8099-73241b49a925
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: d1d43f11c1a90456b24f02a5ec43982d5fdc3de7
ms.sourcegitcommit: 77afc94755db65a3ec107640069067172f55da67
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98694515"
---
# <a name="upload-files-into-an-azure-media-services-account-from-azure-storsimple"></a>從 Azure StorSimple 將檔案上傳至 Azure 媒體服務帳戶 

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> 媒體服務 v2 不會再新增任何新的特性或功能。 <br/>查看最新版本的[媒體服務 v3](../latest/index.yml)。 另請參閱[從 v2 變更為 v3 的移轉指導方針](../latest/migrate-v-2-v-3-migration-introduction.md)
>
> 
> Azure StorSimple Data Manager 目前處於私人預覽階段。 
> 

## <a name="overview"></a>概觀

在媒體服務中，您會將數位檔案上傳到到資產。 資產可以包含影片、音訊、影像、縮圖集合、文字播放軌和隱藏式輔助字幕檔案 (以及這些檔案的相關中繼資料。 ) 檔案上傳之後，您的內容就會安全地儲存在雲端，以進一步處理和串流處理。

[Azure StorSimple](../../storsimple/index.yml) 使用雲端儲存體做為內部部署解決方案的擴充功能，並且跨內部部署儲存體和雲端儲存體自動將資料分層。 StorSimple 裝置會先刪除重複資料並加以壓縮，再將資料傳送至雲端，以非常有效率的方式將大型檔案傳送至雲端。 [StorSimple Data Manager](../../storsimple/storsimple-data-manager-overview.md) 服務提供 API，可讓您從 StorSimple 擷取資料並將它呈現為 AMS 資產。

## <a name="get-started"></a>開始使用

1. [建立媒體服務帳戶](media-services-portal-create-account.md)以便將資產傳輸至其中。
2. 註冊 Data Manager 預覽版本，如 [StorSimple Data Manager](../../storsimple/storsimple-data-manager-overview.md)一文所述。
3. 建立 StorSimple Data Manager 帳戶。
4. 建立資料轉換作業，以在執行時從 StorSimple 裝置擷取資料並將它傳輸到 AMS 帳戶中做為資產。 

    當作業開始執行時，會建立儲存體佇列。 此佇列中會填入已轉換的 blob 備妥時的相關訊息。 此佇列的名稱與作業定義的名稱相同。 您可以使用此佇列來判斷資產何時準備就緒，並呼叫您所需的媒體服務作業以在其上執行。 例如，您可以使用此佇列來觸發 Azure Function，其中有所需的媒體服務程式碼。

## <a name="see-also"></a>另請參閱

[使用 .NET SDK 來觸發資料管理員中的作業](../../storsimple/storsimple-data-manager-dotnet-jobs.md)

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟

您現在可以將上傳的資產編碼。 如需詳細資訊，請參閱 [為資產編碼](media-services-portal-encode.md)。
