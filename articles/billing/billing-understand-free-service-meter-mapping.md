---
title: Azure 免費帳戶之服務與計量的對應 | Microsoft Docs
description: 對於免費帳戶隨附的服務了解服務與計量的對應。
services: ''
documentationcenter: ''
author: amberbhargava
manager: amberb
editor: ''
tags: billing
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/25/2017
ms.author: amberb
ms.openlocfilehash: 3019fe5f8b44eaa32d4ab309f296d6b4574df1d3
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37061365"
---
# <a name="understand-free-service-to-meter-mapping"></a>了解免費帳戶與計量的對應

每個 Azure 服務會發出遵循計量測量的使用量，Azure 計費系統會使用該使用量向使用者收取服務的費用。 若要更加了解免費服務的使用量，可對於這些服務查看服務與計量的對應。 若要了解如何建立免費服務，請參閱[使用 Azure 免費帳戶來建立免費服務](billing-create-free-services-included-free-account.md)。

## <a name="service-to-meter-mapping-for-free-account-eligible-services"></a>免費帳戶適用服務與計量的對應 

|    服務   | Azure 入口網站上的計量名稱 | 使用量檔案/API 中的計量名稱 | 計量識別碼 |
| ------------ | -------------------------- | -------------------------| -------- |
| B1S Linux VM | 計算時數 - Standard_B1 VM | 計算時數 - 免費 | 8260cba2-4437-47d1-a31e-2561cd370f50
| B1S Windows VM | 計算時數 - Standard_B1 VM (Windows) | 計算時數 - 免費 | ff3e6fa5-ee46-478e-8d0e-b629f4f8a8ac
| B1S VM - 公用 IP 位址  | IP 位址時數 - 公用 IP 位址 | IP 位址時數 - 免費 | ae56b367-2708-4454-a3d9-2be7b2364ea1
| CosmosDB | 儲存體 (GB) - Cosmos DB | 儲存體 (GB) - 免費 | 59c78b09-08e2-466a-9f3b-57a94c9e2f31
| CosmosDB | 100 個要求單位 (時數) - Cosmos DB | 100 個要求單位 (時數) - 免費 | 5d638a6f-e221-41cf-ae3f-0f81d368cef6 
| 檔案儲存體 | 標準 IO - 檔案 (GB) - 本地備援 | 標準 IO - 檔案 (GB) - 免費 | a7f2aa67-b9a2-4593-a413-6ec86d6c8e5b
| 檔案儲存體 | 標準 IO - 檔案讀取作業單元 (以萬為單位) | 標準 IO - 檔案讀取作業單元 (以萬為單位) - 免費 | 6207404d-3389-4d20-9087-cc078ddc3fd9
| 檔案儲存體 | 標準 IO - 檔案寫入作業單元 (以萬為單位) | 標準 IO - 檔案寫入作業單元 (以萬為單位) - 免費 | 223d8004-d29a-46cf-b4f4-d2d34b12548b
| 檔案儲存體 | 標準 IO - 檔案通訊協定作業單元 (以萬為單位) | 標準 IO - 檔案通訊協定作業單元 (以萬為單位) - 免費 | a347d8cc-51d1-4a0e-b9eb-76f67566c3f5
| 檔案儲存體 | 標準 IO - 檔案清單作業單元 (以萬為單位) | 標準 IO - 檔案清單作業單元 (以萬為單位) - 免費 | e8ae79ad-c2ab-4d82-b226-dd3c33dfd40c
| 經常性區塊 Blob 儲存體 | 標準 IO - 經常性區塊 Blob 讀取作業 (以萬為單位) | 標準 IO - 經常性區塊 Blob 讀取作業 (以萬為單位) - 免費 |fd7cfa1e-026e-4be1-871b-1c2386e8902e
| 經常性區塊 Blob 儲存體 | 標準 IO - 經常性區塊 Blob (GB) - 本機備援 | 標準 IO - 經常性區塊 Blob (GB) - 免費 | 67a3a3fd-826f-42c1-8843-bffa14f0da13
| 經常性區塊 Blob 儲存體 | 標準 IO - 經常性區塊 Blob 寫入作業 (以萬為單位) | 標準 IO - 經常性區塊 Blob 寫入作業 (以萬為單位) - 免費 | b34bbb76-edce-4c2d-a288-81a2db1fea53
| 經常性區塊 Blob 儲存體  | 標準 IO - 經常性區塊 Blob 寫入/列出作業 (以萬為單位) | 標準 IO - 經常性區塊 Blob 寫入/列出作業 (以萬為單位) - 免費 | 7e68cf36-1198-4d3b-baa7-86a74c5b3079
| 受控磁碟*  | 標準受控磁碟/快照集 (GB) - 本機備援 | 標準受控磁碟/快照集 (GB) - 免費 | ad94c237-52a5-4804-ae65-38c5bf85ef42
| 受控磁碟*  | 標準受控磁碟作業 (以萬為單位) | 標準受控磁碟作業 (以萬為單位) - 免費 | 82cc6ea4-0abd-43ac-acc0-ec34edf0f14c
| 受控磁碟*  | 進階儲存體 - 分頁 Blob/P6 (單位) - 本機備援 | 進階儲存體 - 分頁 Blob/P6 (單位) - 免費 | 2b98c168-27ca-4cc1-b509-e887dec87657
| SQL Database | 標準 S0 資料庫天數 - SQL 資料庫 | 標準 S0 資料庫天數 - 免費 | dd6b69d3-9be0-4a91-abff-2c58bbcafd1d
| 共用 - 頻寬 ** | 資料傳出 (GB) | 資料傳出 (GB) - 免費 | 0fc067a1-65d2-46da-b24b-7a9cbe2c69bd

\* 如果您建立 Windows 虛擬機器，並選擇受控磁碟，您將取用屬於虛擬機器一部分的受控磁碟計量。

\** 可透過多個服務取用共用計量。 例如，虛擬機器和儲存體遵循資料傳出 (GB) 計量發出使用量。





## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
