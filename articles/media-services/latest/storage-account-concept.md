---
title: Azure 媒體服務中的儲存體帳戶 | Microsoft Docs
description: 本文討論媒體服務如何使用儲存體帳戶。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: dbd8dea32bbd4cacefd64e91541ab20b1056731e
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49379022"
---
# <a name="storage-accounts"></a>儲存體帳戶

建立媒體服務帳戶時，您需要提供 Azure 儲存體帳戶資源的名稱。 指定的儲存體帳戶會附加到您的媒體服務帳戶。 

您只能有一個**主要**儲存體帳戶，而與您的媒體服務帳戶相關聯的**次要**儲存體帳戶可以有任意數量。 媒體服務支援**一般用途 v2** (GPv2) 或**一般用途 v1** (GPv1) 帳戶。 

>[!NOTE]
> 僅有 Blob 的帳戶不允許作為**主要**帳戶。 

我們建議您使用 GPv2，這樣便能在經常性存取層與非經常性存取層之間做選擇。 若要深入了解儲存體帳戶，請參閱 [Azure 儲存體帳戶概觀](../../storage/common/storage-account-overview.md)。 

## <a name="assets-in-a-storage-account"></a>儲存體帳戶中的資產

在媒體服務 v3 中，儲存體 API 可用來將檔案上傳。 要想知道如何使用儲存體 API 搭配媒體服務來上傳您的輸入檔案，請參閱[從本機檔案建立作輸入](job-input-from-local-file-how-to.md)。 

> [!Note]
> blob 容器若是由媒體服務 SDK 產生時，請務必使用媒體服務 API 變更 blob 容器的內容。

## <a name="next-steps"></a>後續步驟

若要深入了解如何將儲存體帳戶附加到媒體服務帳戶，請參閱[建立帳戶](create-account-cli-quickstart.md)。
