---
title: 包含檔案
description: 包含檔案
services: azure-policy
author: craigshoemaker
ms.service: azure-policy
ms.topic: include
ms.date: 05/18/2018
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: e6a0ded137162328fd446b65ddb4a15fa6f1db88
ms.sourcegitcommit: 638599eb548e41f341c54e14b29480ab02655db1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36313875"
---
## <a name="deleting-personal-information"></a>刪除個人資訊

[!INCLUDE [gdpr-intro-sentence.md](gdpr-intro-sentence.md)]

個人資訊是關於匯入和匯出作業期間的匯入/匯出服務 (透過入口網站和 API)。 這些程序期間所使用的資料包括：

- 連絡人姓名
- 電話號碼
- 電子郵件
- 街道地址
- City
- 郵遞區號
- State
- 國家/地區/州/省/區域
- 磁碟機識別碼
- 貨運公司帳戶號碼
- 運送追蹤號碼

建立匯入/匯出作業時，使用者會提供連絡人資訊和運送地址。 個人資訊最多會儲存在兩個不同的位置：在作業中，以及選擇性地在入口網站設定中。 如果您在匯出程序的「傳回出貨資訊」區段期間勾選標記 [依預設儲存貨運公司並傳回地址] 的核取方塊，則個人資訊只會儲存在入口網站設定中。

個人連絡人資訊能以下列方式刪除：

- 與作業一起儲存的資料會與作業一起刪除。 使用者可以手動刪除作業，且已完成的作業會在 90 天後自動刪除。 您可以透過 REST API 或 Azure 入口網站手動刪除作業。 若要刪除 Azure 入口網站中的作業，請移至您的匯入/匯出作業，並從命令列中按一下 [刪除]。 如需有關如何透過 REST API 將匯入/匯出作業刪除的詳細資料，請參閱[刪除匯入/匯出作業](../articles/storage/common/storage-import-export-cancelling-and-deleting-jobs.md)。

- 您可以刪除入口網站設定，將儲存在入口網站設定中的連絡人資訊移除。 您可以依照下列步驟來刪除入口網站設定：
  - 登入 [Azure 入口網站](https://portal.azure.com)。
  - 按一下 [設定] 圖示 ![Azure 設定圖示](media/storage-import-export-delete-personal-info/azure-settings-icon.png)
  - 按一下 [匯出所有設定] (將您目前的設定儲存至 `.json` 檔案)。
  - 按一下 [刪除所有設定和私人儀表板] 以刪除所有設定，包括已儲存的連絡人資訊。

如需詳細資訊，請在[信任中心](https://www.microsoft.com/trustcenter)檢閱 Microsoft 隱私權原則