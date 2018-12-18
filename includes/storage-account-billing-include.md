---
title: 包含檔案
description: 包含檔案
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 09/11/2018
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: f47146cef5a81476e76ee5bde3990fac1323148c
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45740583"
---
我們會根據您的儲存體帳戶使用量對「Azure 儲存體」計費。 儲存體帳戶中的所有物件會做為群組共同計費。 

計算儲存體成本時會根據下列因素：區域/位置、帳戶類型、存取層、儲存體容量、複寫配置、儲存體交易及資料輸出。

* **「區域」** 係指您帳戶所在的地理區域。
* 「帳戶類型」係指您目前使用的儲存體帳戶類型。 
* 「存取層」係指您已為一般用途 v2 或 Blob 儲存體帳戶指定的資料使用模式。
* 儲存體「容量」係指您用於儲存資料的儲存體帳戶配額。
* 「複寫」會決定一次維護多少個資料複本，以及在哪些位置維護。
* 「交易」係指對「Azure 儲存體」進行的所有讀取和寫入作業。
* 「資料輸出」係指從 Azure 區域傳出的任何資料。 當您儲存體帳戶中的資料受不同區域中執行的應用程式存取時，您需負擔資料輸出的費用。 如需了解如何使用資源群組來將您的資料和服務聚集在相同區域以限制輸出費用，請參閱[什麼是 Azure 資源群組?](https://docs.microsoft.com/azure/architecture/cloud-adoption/getting-started/azure-resource-access#what-is-an-azure-resource-group) 

[Azure 儲存體價格](https://azure.microsoft.com/pricing/details/storage/) 頁面提供了以帳戶類型、儲存體容量、複寫和交易為基礎的詳細價格資訊。 [資料傳輸定價詳細資料](https://azure.microsoft.com/pricing/details/data-transfers/) 則提供了出口流量的詳細定價資訊。 您可以使用 [Azure 儲存體定價計算機](https://azure.microsoft.com/pricing/calculator/?scenario=data-management) ，以協助消除成本。

