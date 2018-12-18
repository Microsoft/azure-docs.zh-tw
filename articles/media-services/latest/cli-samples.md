---
title: Azure CLI 範例 - Azure 媒體服務 | Microsoft Docs
description: 「Azure 媒體服務」服務的 Azure CLI 範例
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: 5559f9055da3a2a852427c0f27d367159cdc7655
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388499"
---
# <a name="azure-cli-examples-for-azure-media-services"></a>Azure 媒體服務的 Azure CLI 範例

下表包含「Azure 媒體服務」的 Azure CLI 範例連結。

|  |  |
|---|---|
|**帳戶**||
| [建立媒體服務帳戶](./scripts/cli-create-account.md) | 建立 Azure 媒體服務帳戶。 此外，建立一個服務主體，並用它來存取 API，然後以程式設計方式管理該帳戶。 |
| [重設帳戶認證](./scripts/cli-reset-account-credentials.md)|重設帳戶認證，並還原為 app.config 設定。|
|**資產**||
| [建立資產](./scripts/cli-create-asset.md)|建立媒體服務資產以供上傳內容。|
| [上傳檔案](./scripts/cli-upload-file-asset.md)|將本機檔案上傳至儲存體容器。|
| [發佈資產](./scripts/cli-publish-asset.md)| 建立串流定位器並取回串流 URL。 |
| **轉換**與**作業**||
| [建立轉換](./scripts/cli-create-transform.md)|示範如何建立轉換。 轉換會說明用於處理視訊或音訊檔案的簡單工作流程 (通常稱為「配方」)。<br/> 請一律要檢查是否已有所需名稱和「配方」的轉換。 若有，請予以重複使用。 |
| [建立作業](./scripts/cli-create-jobs.md)|使用 HTTPs URL 提交作業給簡單的編碼轉換。|
| [建立 EventGrid](./scripts/cli-create-event-grid.md)|針對作業狀態變更建立帳戶層級的事件方格訂用帳戶。|

## <a name="see-also"></a>另請參閱

[Azure CLI](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)
