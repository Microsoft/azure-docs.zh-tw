---
title: 以最短停機時間移轉至適用於 PostgreSQL 的 Azure 資料庫
description: 本文說明如何使用 Azure 資料庫移轉服務，以最短的停機時間執行從 PostgreSQL 資料庫到「適用於 PostgreSQL 的 Azure 資料庫」的移轉。
services: postgresql
author: HJToland3
ms.author: jtoland
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 06/21/2018
ms.openlocfilehash: 9ab5d4615a8baf763d0b7ee47bf0890124f8665c
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36292537"
---
# <a name="minimal-downtime-migration-to-azure-database-for-postgresql"></a>以最短停機時間移轉至適用於 PostgreSQL 的 Azure 資料庫
您可以使用 [Azure 資料庫移轉服務](https://aka.ms/get-dms) (DMS) 新導入的**持續同步功能**，以最短的停機時間執行從 PostgreSQL 到「適用於 PostgreSQL 的 Azure 資料庫」的移轉。 這項功能可縮短應用程式所造成的停機時間。

## <a name="overview"></a>概觀
DMS 會執行「適用於 PostgreSQL 的 Azure 資料庫」內部部署的初始載入，然後在應用程式仍在執行時持續將任何新交易同步至 Azure。 當資料同步至目標 Azure 端後，您應短暫停止應用程式 (最短停機時間)，等待最後一批資料 (從您停止應用程式，到應用程式實際上無法再接收任何新流量為止) 同步至目標端，然後再更新連接字串以指向 Azure。 當您完成時，應用程式將會在 Azure 上執行！

![透過 Azure 資料庫移轉服務進行持續同步](./media/howto-migrate-online/ContinuousSync.png)

PostgreSQL 來源的 DMS 移轉目前處於預覽階段。 如果您想要嘗試以此服務移轉 PostgreSQL 工作負載，請透過 Azure DMS [預覽頁面](https://aka.ms/dms-preview)註冊以表達您的意願。 您的意見反應將是進一步改善此服務的重要助力。

## <a name="next-steps"></a>後續步驟
- 檢視[使用 Microsoft Azure 將應用程式現代化](https://medius.studios.ms/Embed/Video/BRK2102?sid=BRK2102)影片，影片中會示範如何將 PostgreSQL 應用程式移轉至適用於 PostgreSQL 的 Azure 資料庫。
- 透過[預覽頁面](https://aka.ms/dms-preview)，註冊以最短的停機時間將 PostgreSQL 移轉至「適用於 PostgreSQL 的 Azure 資料庫」的有限預覽版移轉。
