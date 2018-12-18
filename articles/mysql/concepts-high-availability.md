---
title: 適用於 MySQL 之 Azure 資料庫中的高可用性概念
description: 本主題提供使用「適用於 MySQL 的 Azure 資料庫」時的高可用性資訊
services: mysql
author: jasonwhowell
ms.author: jasonh
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 90dc603c0ee520774bd22531c7136e0949f6cf90
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35264175"
---
# <a name="high-availability-concepts-in-azure-database-for-mysql"></a>適用於 MySQL 之 Azure 資料庫中的高可用性概念
「適用於 MySQL 的 Azure 資料庫」服務可提供保證的高可用性等級。 以財務為後盾的服務等級協定 (SLA) 在正式運作時可達 99.99%。 使用此服務時幾乎不會有應用程式停機時間。

## <a name="high-availability"></a>高可用性
高可用性 (HA) 模型依據的是發生節點層級中斷時的內建容錯移轉機制。 之所以發生節點層級中斷，可能是因為硬體故障，或為了回應服務部署。

不論何時，對「適用於 MySQL 的 Azure 資料庫」資料庫伺服器所做的變更都會在交易內容中進行。 這些變更會在認可交易時在 Azure 儲存體中同步記錄下來。 如果發生節點層級中斷，資料庫伺服器會自動建立新節點，然後將資料儲存體連結至該新節點。 系統會捨棄所有作用中的連線，並且不會認可任何進行中的交易。

## <a name="application-retry-logic-is-essential"></a>應用程式重試邏輯相當重要
請注意，MySQL 資料庫應用程式的建置目的是要偵測並重試被捨棄的連線，以及失敗的交易。 當應用程式進行重試時，應用程式的連線會在背景中重新導向到新建立的執行個體，此執行個體會接替失敗的執行個體。

在 Azure 內部，會使用閘道將連線重新導向到新的執行個體。 發生中斷時，整個容錯移轉程序通常需要花費數十秒鐘的時間。 由於重新導向是由閘道在內部處理的，因此用戶端應用程式的外部連接字串會保持不變。

## <a name="scaling-up-or-down"></a>擴大或縮小規模
與 HA 模型類似，當「適用於 MySQL 的 Azure 資料庫」擴大或縮小規模時，會以指定的大小建立新的伺服器執行個體。 現有的資料儲存體會與原始執行個體中斷連結，然後連結到新的執行個體。

進行規模調整作業時，會發生資料庫連線中斷。 系統會將用戶端應用程式中斷連線，並將已開啟的未認可交易取消。 一旦用戶端應用程式重試連線或建立新連線，閘道就會將該連線導向到新調整過大小的執行個體。 

## <a name="next-steps"></a>後續步驟
- 如需服務的概觀，請參閱[適用於 MySQL 的 Azure 資料庫概觀](overview.md)
