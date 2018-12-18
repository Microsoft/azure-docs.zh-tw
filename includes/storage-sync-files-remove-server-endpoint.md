---
title: 包含檔案
description: 包含檔案
services: storage
author: wmgries
ms.service: storage
ms.topic: include
ms.date: 05/31/2018
ms.author: wgries
ms.custom: include file
ms.openlocfilehash: 33dff710d83bd12a8db343a89c6e4576d1397ba7
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38763072"
---
否：移除伺服器端點不像是重新啟動伺服器！ 要解決同步處理、雲端層或其他方面的 Azure 檔案同步等問題，將伺服器端點移除然後再加以重新建立通常不是適當的解決方案。將伺服器端點移除是一種破壞性作業，且在分層的檔案位於伺服器端點命名空間外部的情況下，可能會導致資料遺失 (如需詳細資訊，請參閱[分層的檔案為何會位於伺服器端點命名空間外部](../articles/storage/files/storage-files-faq.md#afs-tiered-files-out-of-endpoint))，或是在分層的檔案位於伺服器端點命名空間內部時，會導致無法存取檔案。 重新建立伺服器端點將不會解決這些問題。 分層的檔案可能會位於您的伺服器端點命名空間內部，即使您從未啟用過雲端分層。 因此，建議您不要將伺服器端點移除，除非您要停止使用與此特定資料夾進行 Azure 檔案同步，或是已收到 Microsoft 工程師的明確指示，要執行這項操作。 如需關於移除伺服器端點的詳細資訊，請參閱[移除伺服器端點](../articles/storage/files/storage-sync-files-server-endpoint.md#remove-a-server-endpoint)。    