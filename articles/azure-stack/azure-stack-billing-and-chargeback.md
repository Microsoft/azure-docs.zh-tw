---
title: Azure Stack 中的客戶帳務與退款 | Microsoft Docs
description: 了解如何從 Azure Stack 擷取資源使用量資訊。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2018
ms.author: sethm
ms.reviewer: alfredop
ms.openlocfilehash: fcfd5d4e9e2f30d769818df29cf8a76bd9113d4f
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45632400"
---
# <a name="usage-and-billing-in-azure-stack"></a>Azure Stack 中的使用量與計費

此文說明向 Azure Stack 使用者收取資源使用費用的方式。 您可以了解如何存取計費資訊以進行分析和退款。

Azure Stack 會收集已使用資源的使用量資料並加以群組。 然後，Azure Stack 會將此資料轉送到 Azure Commerce。 Azure Commerce 對 Azure Stack 使用量的計費方式與 Azure 使用量相同。

您也可以使用計費配接器來取得使用量資料，並匯出到您自己的計費或退款系統，或是匯出到商業智慧工具 (例如 Microsoft Power BI)。


## <a name="usage-pipeline"></a>使用量管線

Azure Stack 中的每個資源提供者都會依據資源使用情況張貼使用量資料。 「使用量服務」會定期 (每小時和每天) 彙總使用量資料，並將其儲存在使用量資料庫中。 Azure Stack 操作員和使用者可以透過「Azure Stack 資源使用量 API」來存取此已儲存的使用量資料。 

如果您已[向 Azure 註冊 Azure Stack 執行個體](azure-stack-register.md)，則 Azure Stack 已設定為將該使用量資料傳送給 Azure Commerce。 將該資料上傳到 Azure 之後，您便可以透過計費入口網站或使用「Azure 資源使用量 API」來存取它。 若要深入了解會向 Azure 回報哪些使用量資料，請參閱[使用量資料回報](azure-stack-usage-reporting.md)一文。  

下圖顯示使用量管線中的主要元件： 

![使用量管線](media\azure-stack-billing-and-chargeback\usagepipeline.png)

## <a name="what-usage-information-can-i-find-and-how"></a>我可以找到哪些使用量資訊，以及如何尋找？

Azure Stack 資源提供者 (例如計算、儲存體及網路) 會以每小時為間隔，針對每個訂用帳戶產生使用量資料。 使用量資料包含與所使用資源有關的資訊，例如資源名稱、所使用的訂用帳戶，以及所使用的數量。 若要了解計量識別碼資源，請參閱[使用量 API 常見問題集](azure-stack-usage-related-faq.md)一文。

收集到使用量資料之後，就會將資料[回報給 Azure](azure-stack-usage-reporting.md)，以產生可透過 Azure 計費入口網站檢視的帳單。 

> [!NOTE]  
> 對「Azure Stack 開發套件」使用者和採用容量授權模型的 Azure Stack 整合系統使用者而言，使用量資料回報並非必要功能。 若要深入了解 Azure Stack 中的授權，請參閱[封裝與定價](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf) \(英文\) 資料表。

Azure 計費入口網站會顯示可收費資源的使用量資料。 除了可收費資源之外，Azure Stack 可擷取更廣泛資源的使用量資料，您可以透過 REST API 或 PowerShell，在 Azure Stack 環境中存取那些資料。 Azure Stack 操作員可以取得所有使用者訂用帳戶的使用量資料。 個別使用者只能取得他們自己的使用量詳細資料。 

## <a name="usage-reporting-for-multitenant-cloud-service-providers"></a>多租用戶雲端服務提供者的使用量回報

多租用戶「雲端服務提供者」(CSP) 若有許多使用 Azure Stack 的客戶，可以個別回報每個客戶的使用量，以便提供者向不同的 Azure 訂用帳戶收取使用費用。 

每個客戶將會有以不同 Active Directory (Azure AD) 租用戶代表的身分識別。 Azure Stack 支援為每個 Azure AD 租用戶各指派一個 CSP 訂用帳戶。 您可以將租用戶及其訂用帳戶新增至基礎 Azure Stack 註冊。 針對所有 Azure Stack 都會進行基礎註冊。 如果沒有為租用戶註冊訂用帳戶，使用者仍然可以使用 Azure Stack，而其使用情況則會傳送給用於基礎註冊的訂用帳戶。 


## <a name="next-steps"></a>後續步驟

[向 Azure 註冊 Azure Stack](azure-stack-registration.md)

[向 Azure 回報 Azure Stack 使用量資料](azure-stack-usage-reporting.md)

[提供者資源使用情況 API](azure-stack-provider-resource-api.md)

[租用戶資源使用情況 API](azure-stack-tenant-resource-usage-api.md)

[使用量相關常見問題集](azure-stack-usage-related-faq.md)