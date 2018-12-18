---
title: 透過入口網站啟用 Azure 事件中樞擷取功能 | Microsoft Docs
description: 使用 Azure 入口網站啟用事件中樞擷取功能。
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: ff80bc2452c9826a5c51c146a957fddc72d2dbc2
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "41920269"
---
# <a name="enable-event-hubs-capture-using-the-azure-portal"></a>使用 Azure 入口網站啟用事件中樞擷取功能

Azure [事件中樞擷取][capture-overview]可讓您將事件中樞內的串流資料自動傳遞至您選擇的 [Azure Blob 儲存體](https://azure.microsoft.com/services/storage/blobs/)或 [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/) 帳戶。

您可以使用 [Azure 入口網站](https://portal.azure.com)，在建立事件中樞時設定擷取功能。 您可以將資料擷取至 Azure [Blob 儲存體](https://azure.microsoft.com/services/storage/blobs/)容器或 [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/) 帳戶。

如需詳細資訊，請參閱[事件中樞擷取概觀][capture-overview]。

## <a name="capture-data-to-an-azure-storage-account"></a>將資料擷取至 Azure 儲存體帳戶  

當您建立事件中樞時，按一下 [建立事件中樞] 入口網站畫面中的 [開啟] 按鈕，即可啟用擷取功能。 然後按一下 [擷取提供者] 方塊中的 [Azure 儲存體]，以指定儲存體帳戶和容器。 事件中樞擷取會對儲存體使用服務對服務驗證，因此您不需要指定儲存體連接字串。 資源選擇器會自動選取儲存體帳戶的資源 URI。 如果您使用 Azure Resource Manager，您必須以字串形式明確提供此 URI。

預設時間範圍為 5 分鐘。 最小值是 1，最大值是 15。 **大小** 範圍為 10-500 MB。

![][1]

## <a name="capture-data-to-an-azure-data-lake-store-account"></a>將資料擷取至 Azure Data Lake Store 帳戶

若要將資料擷取至 Azure Data Lake Store，請建立 Data Lake Store 帳戶及事件中樞：

### <a name="create-an-azure-data-lake-store-account-and-folders"></a>建立 Azure Data Lake Store 帳戶和資料夾

1. 遵循[使用 Azure 入口網站開始使用 Azure Data Lake Store](../data-lake-store/data-lake-store-get-started-portal.md) 中的指示，建立 Data Lake Store 帳戶。
2. 請依照[將權限指派給事件中樞](../data-lake-store/data-lake-store-archive-eventhub-capture.md#assign-permissions-to-event-hubs)一節中的指示，在您要從事件中樞擷取資料的 Data Lake Store 帳戶中建立資料夾，並將權限指派給事件中樞，以便將資料寫入 Data Lake Store 帳戶中。  

### <a name="create-an-event-hub"></a>建立事件中樞

1. 請注意，事件中樞必須位於與您剛建立之 Azure Data Lake Store 相同的 Azure 訂用帳戶中。 按一下 [建立事件中樞] 入口網站頁面中 [擷取] 之下的 [開啟] 按鈕，以建立事件中樞。 
2. 在 [建立事件中樞] 入口網站頁面中，從 [擷取提供者] 方塊選取 [Azure Data Lake Store]。
3. 在 [選取 Data Lake Store] 中，指定您先前建立的 Data Lake Store 帳戶，以及在 [Data Lake 路徑] 欄位中，輸入您建立之資料資料夾的路徑。

    ![][3]

## <a name="add-or-configure-capture-on-an-existing-event-hub"></a>在現有的事件中樞上新增或設定擷取功能

您可以在位於事件中樞命名空間的現有事件中樞上設定擷取功能。 若要在現有事件中樞上啟用擷取功能，或變更您的擷取設定，請按一下命名空間以載入概觀畫面，然後按一下您要啟用或變更擷取設定的事件中樞。 最後，按一下開啟之頁面左側的 [擷取] 選項，然後編輯設定，如下圖所示：

### <a name="azure-blob-storage"></a>Azure Blob 儲存體

![][2]

### <a name="azure-data-lake-store"></a>Azure Data Lake Store

![][4]

[1]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture1.png
[2]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture2.png
[3]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture3.png
[4]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture4.png

## <a name="next-steps"></a>後續步驟

- 閱讀[事件中樞擷取概觀][capture-overview]，深入了解事件中樞擷取功能。
- 您也可以透過 Azure Resource Manager 範本來設定事件中樞擷取。 如需詳細資訊，請參閱[使用 Azure Resource Manager 範本啟用擷取功能](event-hubs-resource-manager-namespace-event-hub-enable-capture.md)。
- [使用 Azure 入口網站開始使用 Azure Data Lake Store](../data-lake-store/data-lake-store-get-started-portal.md)

[capture-overview]: event-hubs-capture-overview.md