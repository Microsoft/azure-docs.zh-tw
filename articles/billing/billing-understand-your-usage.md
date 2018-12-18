---
title: 了解您的 Azure 詳細使用量 | Microsoft Docs
description: 了解如何閱讀並了解 Azure 訂用帳戶的詳細使用量 CSV 區段
services: ''
documentationcenter: ''
author: tonguyen10
manager: tonguyen
editor: ''
tags: billing
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/31/2017
ms.author: tonguyen
ms.openlocfilehash: 723e42d2bb2af09eb2236c3cbefeee33987ea45b
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37060795"
---
# <a name="understand-terms-on-your-microsoft-azure-detailed-usage-charges"></a>了解您 Microsoft Azure 詳細使用量費用的相關字詞 

詳細的使用量費用 CSV 檔案會包含目前計費期間的每日及計量層級使用量費用。 

若要取得詳細使用量檔案，請參閱[如何取得您的 Azure 帳單發票和每日使用量資料](billing-download-azure-invoice-daily-usage-date.md)。
此檔案以逗號分隔值 (.csv) 檔案格式提供，您可以使用試算表應用程式開啟這個檔案。 如果您看到兩個可用版本，請下載第 2 個版本。 這是最新的檔案格式。

使用量費用是訂用帳戶的**每月**總費用。 使用量費用不會將任何信用額度或折扣列入考慮。

>[!VIDEO https://www.youtube.com/embed/p13S350M2Vk]

## <a name="detailed-terms-and-descriptions-of-your-detailed-usage-file"></a>詳細使用量檔案的詳細字詞和描述

下列各節說明第 2 版詳細使用量檔案中所示的重要字詞。

### <a name="statement"></a>陳述式

詳細使用量 CSV 檔案的上方區段會顯示您在當月計費期間使用的服務。 下表列出本節中所顯示的詞彙和說明。

| 詞彙 | 說明 |
| --- | --- |
|計費期間 |使用計量時的計費期間 |
|計量類別 |識別使用量的最上層服務 |
|計量子類別 |定義可能會影響費率的 Azure 服務類型 |
|計量名稱 |識別耗用計量的量值單位 |
|計量區域 |針對以資料中心位置作為基礎所定價的某些服務，識別資料中心的位置 |
|SKU |識別每個 Azure 計量的唯一系統識別碼 |
|單位 |識別服務的計費單位。 例如，GB、小時、10,000 秒。 |
|已耗用的數量 |計費期間所使用之計量的數量 |
|已包括的數量 |目前計費期間免費提供之計量的數量 |
|超額數量 |顯示「已耗用的數量」和「已包括的數量」之間的差額。 我們會針對此數量向您收費。 對於不提供供應項目所含數量的隨收隨付供應項目，這個總數會與「已耗用的數量」相同。 |
|承諾用量期間 |顯示從您的 6 或 12 個月供應項目相關認可用量中減去的計量費用。 計量費用得依時間順序減去。 |
|貨幣 |目前計費期間所使用的貨幣 |
|超額部分 |顯示超過您的 6 或 12 個月供應項目相關認可用量的計量費用 |
|承諾用量費率 |以您的 6 或 12 個月供應項目相關聯總認可用量作為基礎所顯示的認可用量費率 |
|費率 |每個計費單位向您收取的費率 |
|值 |顯示 [超額數量] 資料行和 [費率] 資料行兩者相乘的結果。 如果「已耗用的數量」沒有超過「已包括的數量」，則此資料行中沒有任何費用。 |

### <a name="daily-usage"></a>每日使用量

CSV 檔案的 [每日使用量] 區段會顯示影響收費率的使用量詳細資料。 下表列出本節中所顯示的詞彙和說明。

| 詞彙 | 說明 |
| --- | --- |
|使用日期 |使用計量時的日期 |
|計量類別 |識別這個使用量所屬的最上層服務 |
|計量識別碼 |用於訂定計費使用量價格的計費計量識別碼 |
|計量子類別 |定義可能會影響費率的 Azure 服務類型 |
|計量名稱 |識別耗用計量的量值單位 |
|計量區域 |針對以資料中心位置作為基礎所定價的某些服務，識別資料中心的位置 |
|單位 |識別計量的計費單位。 例如，GB、小時、10,000 秒。 |
|已耗用的數量 |當日已耗用的計量量 |
|資源位置 |識別正在執行計量的資料中心 |
|已耗用的服務 |您所使用的 Azure 平台服務 |
|資源群組 |部署的資源正在其中執行的計量群組。 <br/><br/>如需詳細資訊，請參閱 [Azure Resource Manager 概觀](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。 |
|執行個體識別碼 | 計量的識別碼。 <br/><br/> 識別碼包含在建立時為計量指定的名稱。 這會是資源的名稱或完整的資源 ID。 如需詳細資訊，請參閱 [Azure Resource Manager API](https://docs.microsoft.com/rest/api/resources/resources)。 |
|標記 | 您指派給計量的標記。 使用標記為帳單記錄分組。<br/><br/>例如，您可以使用標記，根據使用計量的部門散發成本。 支援發出標記的服務為虛擬機器、儲存體，以及使用 [Azure Resource Manager API](https://docs.microsoft.com/rest/api/resources/resources) 佈建的網路服務。 如需詳細資訊，請參閱[使用標記組織您的 Azure 資源](http://azure.microsoft.com/updates/organize-your-azure-resources-with-tags/)。 |
|其他資訊 |服務專屬的中繼資料。 例如，虛擬機器的影像類型。 |
|服務資訊 1 |服務在訂用帳戶上所屬的專案名稱 |
|服務資訊 2 |舊版欄位，可擷取選擇性的服務特定中繼資料 |

## <a name="how-do-i-make-sure-that-the-charges-in-my-detailed-usage-file-are-correct"></a>如何確定我詳細使用量檔案中的費用是否正確？
如果您需要詳細使用量檔案費用的更多詳細資料，請參閱[了解您的 Microsoft Azure 帳單。](./billing-understand-your-bill.md)

## <a name="external"></a>外部服務費用呢？
外部服務 (也稱為 Marketplace 訂單) 是由獨立服務廠商提供，並會分開計費。 費用不會顯示在 Azure 發票上。 若要深入了解，請參閱[了解您的 Azure 外部服務費用](billing-understand-your-azure-marketplace-charges.md)。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。
如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?)以快速解決您的問題。
