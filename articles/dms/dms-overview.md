---
title: Azure 資料庫移轉服務概觀 | Microsoft Docs
description: Azure 資料庫移轉服務的概觀，此服務能從許多資料庫來源提供無縫移轉到 Azure 資料平台。
services: database-migration
author: HJToland3
ms.author: jtoland
manager: ''
ms.reviewer: douglasl
ms.service: database-migration
ms.workload: data-services
ms.topic: article
ms.date: 09/01/2018
ms.openlocfilehash: d59850b0234912b02b003f4fc8089d76130151ba
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666224"
---
# <a name="what-is-the-azure-database-migration-service"></a>什麼是 Azure 資料庫移轉服務？
Azure 資料庫移轉服務是一個完全受控的服務，能夠從多個資料庫來源無縫移轉到 Azure 資料平台，將停機時間降到最低。

## <a name="migrate-databases-to-azure-with-familiar-tools"></a>使用熟悉的工具來將資料庫移轉到 Azure
Azure 資料庫移轉服務整合我們現有工具和服務的某些功能。 它提供客戶全方位、高可用性的解決方案。 服務會使用[資料移轉小幫手](http://aka.ms/dma)來產生評估報表，提供建議以引導您在移轉之前完成所需的變更。 由您自行決定，是否要執行任何所需的補救。 當您準備好要開始移轉程序時，Azure 資料庫移轉服務會執行所有必要步驟。 移轉程序會善用 Microsoft 決定的最佳做法，因此您可以放心地移轉專案。

## <a name="regional-availability"></a>區域可用性
Azure 資料庫移轉服務目前可以在下列區域使用：

![Azure 資料庫移轉服務區域可用性](media\overview\dms-regional-availability.png)

> [!NOTE]
> 線上移轉與 SKU 建議功能目前僅適用於 [美國中部][美國東部 2] 及 [西歐] 區域。

如需 Azure 全域基礎結構網站上 Azure 資料庫移轉服務區域可用性的最新資訊，請參閱[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。

## <a name="next-steps"></a>後續步驟
- [使用 Azure 入口網站建立 Azure 資料庫移轉服務的執行個體](quickstart-create-data-migration-service-portal.md)。
- [將 SQL Server 移轉到 Azure SQL Database](tutorial-sql-server-to-azure-sql.md)。
- [使用 Azure 資料庫移轉服務的必要條件概觀](pre-reqs.md)。
- [使用 Azure 資料庫移轉服務的相關常見問題集](faq.md)。
