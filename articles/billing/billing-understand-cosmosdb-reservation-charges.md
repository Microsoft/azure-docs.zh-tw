---
title: 了解保留折扣如何套用至 Azure Cosmos DB | Microsoft Docs
description: 了解保留折扣如何套用至 Azure Cosmos DB 中已佈建的輸送量 (RU/秒)。
services: cosmos-db
author: rimman
manager: kfile
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: rimman
ms.reviewer: sngun
ms.openlocfilehash: 8c5e9c8437c516a5e6d1d97494b21bfc8e90d88e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960255"
---
# <a name="understand-how-the-reservation-discount-is-applied-to-azure-cosmos-db"></a>了解保留折扣如何套用至 Azure Cosmos DB

購買 Azure Cosmos DB 保留容量之後，保留折扣會自動套用至符合保留屬性和數量的 Azure Cosmos DB 資源。 保留會涵蓋為 Azure Cosmos DB 資源佈建的輸送量，而不會涵蓋軟體、網路、儲存體或預先定義的容器費用。

## <a name="reservation-discount-applied-to-azure-cosmos-db-accounts"></a>套用至 Azure Cosmos DB 帳戶的保留折扣

保留折扣會以每小時為基礎，以每秒要求單位 (RU/秒) 套用至[已佈建的輸送量](../cosmos-db/request-units.md)。 針對執行時間不滿一小時的 Azure Cosmos DB 資源，保留折扣會自動套用至其他符合保留屬性的 Cosmos DB 資源。 此折扣可套用至同時執行的 Azure Cosmos DB 資源。 如果沒有既執行滿一個小時又符合保留屬性的 Cosmos DB 資源，您就無法獲得該小時保留折扣的完整權益。

* 折扣會分層，也就是說要求單位越高的保留提供的折扣也越高。  
* 折扣購買會依照與區域隨選價格相等的比例，將折扣套用至所有區域。 如需了解每個區域中的保留折扣比例，請參閱本文的[每一區域的保留折扣](#reservation-discount-per-region)一節。

## <a name="reservation-discount-per-region"></a>每一區域的保留折扣
保留折扣會以每小時為基礎，在單一訂用帳戶或已註冊/帳戶範圍，套用至 Azure Cosmos DB 輸送量成本。 保留折扣會依下列比例套用至不同區域中的計量使用量：

|計量描述  |區域 |比例  |
|---------|---------|---------|
|Azure Cosmos DB - 100 RU/秒/小時 - 亞太地區東南部  |  亞太地區東南部    |   1      |
|Azure Cosmos DB - 100 RU/秒/小時 - 亞太地區東部 |   亞太地區東部   |    1     |
|Azure Cosmos DB - 100 RU/秒/小時 - 歐洲北部|  歐洲北部       |    1     |
|Azure Cosmos DB - 100 RU/秒/小時 - 南韓南部|    南韓南部     |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 歐洲西部|    歐洲西部     |      1   |
|Azure Cosmos DB - 100 RU/秒/小時 - 南韓中部|   南韓中部    |       1  |
|Azure Cosmos DB - 100 RU/秒/小時 - 英國南部|   英國南部      |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 英國西部|   英國西部      |    1     |
|Azure Cosmos DB - 100 RU/秒/小時 - 英國北部 |   英國北部    |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 英國南部 2|   英國南部 2      |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國東部 2|  美國東部 2     |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國中北部|   美國中北部      |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國西部|   美國西部      |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國中部| 美國中部        |     1    |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國西部 2|   美國西部 2      |      1   |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國中西部|   美國中西部      |       1  |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國東部|   美國東部      |  1       |
|Azure Cosmos DB - 100 RU/秒/小時 - 南非北部|     南非北部    |   1      |
|Azure Cosmos DB - 100 RU/秒/小時 - 南非西部 |    南非西部      |    1     |
|Azure Cosmos DB - 100 RU/秒/小時 - 印度南部|    印度南部     |    1.0375    |
|Azure Cosmos DB - 100 RU/秒/小時 - 加拿大東部|   加拿大東部      |    1.1      |
|Azure Cosmos DB - 100 RU/秒/小時 - 日本東部|   日本東部      |    1.125     |
|Azure Cosmos DB - 100 RU/秒/小時 - 日本西部|     日本西部    |   1.125       |
|Azure Cosmos DB - 100 RU/秒/小時 - 印度西部|     印度西部    |    1.1375     |
|Azure Cosmos DB - 100 RU/秒/小時 - 印度中部|    印度中部     |  1.1375       |
|Azure Cosmos DB - 100 RU/秒/小時 - 澳大利亞東部|     澳大利亞東部    |   1.15       |
|Azure Cosmos DB - 100 RU/秒/小時 - 加拿大中部|  加拿大中部       |   1.2       |
|Azure Cosmos DB - 100 RU/秒/小時 - 法國中部|   法國中部      |    1.25      |
|Azure Cosmos DB - 100 RU/秒/小時 - 巴西南部|  巴西南部       |   1.5      |
|Azure Cosmos DB - 100 RU/秒/小時 - 澳大利亞中部|   澳大利亞中部      |   1.5      |
|Azure Cosmos DB - 100 RU/秒/小時 - 澳大利亞中部 2| 澳大利亞中部 2        |    1.5     |
|Azure Cosmos DB - 100 RU/秒/小時 - 法國南部|    法國南部     |    1.625     |

## <a name="scenarios-that-show-how-the-reservation-discount-is-applied"></a>說明保留折扣套用方式的案例

請思考一下下列保留需求：

* 所需的輸送量： 50,000 RU/秒  
* 使用的區域：2。 

在此案例中，您的需求費用總計會針對這兩個區域中的 500 個 100 RU/秒計量，每小時總計會耗用 100,000 RU/秒。 

**案例 1**

例如，假設您需要「美國中北部」和「美國西部」區域中的 Azure Cosmos DB 部署，而每個區域的輸送量耗用量為 50,000 RU/秒。 購買 100,000 RU/秒的保留將可完全平衡您的需求費用。

保留所涵蓋折扣的計算方式為 (輸送量耗用量 * 該區域的保留折扣比例)。 就「美國中北部」和「美國西部」區域而言，保留折扣比例為 "1"。 因此，總共折扣的 RU/秒為 100,000 RU/秒 (此值的計算方式為：50,000 * 1 + 50,000 * 1 = 100,000 RU/秒)，且您無須支付任何其他預付型方案費率的費用。 

|計量描述 | 區域 |輸送量耗用量 (RU/秒) |套用至 RU/秒的保留折扣 |
|---------|---------|---------|---------|
|Azure Cosmos DB - 100 RU/秒/小時 - 美國中北部  |   美國中北部  | 50,000  | 50,000  |
|Azure Cosmos DB - 100 RU/秒/小時 - 美國西部  |  美國西部   |  50,000  |  50,000 |

**案例 2**

例如，假設您需要「澳大利亞中部 2」和「法國南部」區域中的 Azure Cosmos DB 部署，而每個區域的輸送量耗用量為 50,000 RU/秒。 購買 100,000 RU/秒的保留時，會以下列方式套用 (假設先對「澳大利亞中部 2」進行折扣)：

|計量描述 | 區域 |輸送量耗用量 (RU/秒) |套用至 RU/秒的保留折扣 |
|---------|---------|---------|---------|
|Azure Cosmos DB - 100 RU/秒/小時 - 澳大利亞中部 2  |  澳大利亞中部 2   |  50,000  |  50,000   |
|Azure Cosmos DB - 100 RU/秒/小時 - 法國南部  |  法國南部   |  50,000 |  15,384  |

「澳大利亞中部 2」中的 50,000 單位使用量相當於 75,000 RU/秒的可計費使用量 (或標準化使用量)。 此值的計算方式為 (輸送量耗用量 * 該區域的保留折扣比例)，這等於 75,000 RU/秒 (此值的計算方式為：50,000 * 1.5 = 75,000 RU/秒) 的可計費或標準化使用量。 

購買 100,000 RU/秒的保留可抵銷「澳大利亞中部 2」中的 75,000 RU/秒，留下 25,000 RU/秒給「法國南部」區域。 從剩餘的 25,000 RU/秒中，會在「法國南部」區域套用 15,384 RU/秒的保留折扣 (此值的計算方式為：25,000 / 1.625 = 15,384 RU/秒)。 「法國南部」區域中剩餘的 34,616 RU/秒會依一般預付型方案費率收費。 

Azure 計費系統會將保留計費權益指派給第一個處理且符合保留設定的執行個體 (例如本案例中的「澳大利亞中部 2」)。

若要在計費使用量報告中了解及檢視 Azure 保留的應用，請參閱[了解 Azure 保留使用量](../billing/billing-understand-reserved-instance-usage-ea.md)。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 保留項目，請參閱下列文章：

* [什麼是 Azure 保留項目？](../billing/billing-save-compute-costs-reservations.md)  
* [以 Azure Cosmos DB 保留容量預先支付 Azure Cosmos DB 資源的費用](../cosmos-db/cosmos-db-reserved-capacity.md)  
* [以 Azure SQL Database 保留容量預先支付 SQL 資料庫計算資源的費用](../sql-database/sql-database-reserved-capacity.md)  
* [管理 Azure 保留項目](../billing/billing-manage-reserved-vm-instance.md)  
* [了解隨用隨付方案訂用帳戶的保留項目使用量](../billing/billing-understand-reserved-instance-usage.md)  
* [了解 Enterprise 註冊的保留項目使用量](../billing/billing-understand-reserved-instance-usage-ea.md)  
* [了解 CSP 訂用帳戶的保留使用量](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。

