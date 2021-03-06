---
title: 包含檔案
description: 包含檔案
services: redis-cache
author: wesmc7777
ms.service: cache
ms.topic: include
ms.date: 11/05/2019
ms.author: wesmc
ms.custom: include file
ms.openlocfilehash: a737e130d616a67bab28c7c96c0372216a6707af
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002441"
---
### <a name="retrieve-host-name-ports-and-access-keys-from-the-azure-portal"></a>從 Azure 入口網站擷取主機名稱、連接埠和存取金鑰

若要連線至 Azure Cache for Redis 執行個體，快取用戶端需要主機名稱、連接埠和快取金鑰。 某些用戶端可能會以稍有不同的名稱來參考這些項目。 您可以從 [Azure 入口網站](https://portal.azure.com)取得主機名稱、連接埠和金鑰。

- 若要取得存取金鑰，請從快取的左側導覽中選取 [存取金鑰]  。 
  
  ![Azure Cache for Redis 金鑰](media/redis-cache-access-keys/redis-cache-keys.png)

- 若要取得主機名稱和連接埠，請從快取的左側導覽中選取 [屬性]  。 主機名稱的格式為 *\<DNS name>.redis.cache.windows.net*。

  ![Azure Redis 快取屬性](media/redis-cache-access-keys/redis-cache-hostname-ports.png)

