---
ms.assetid: ''
title: Azure Key Vault 安全世界 | Microsoft Docs
ms.service: key-vault
ms.topic: conceptual
author: bryanla
ms.author: bryanla
manager: mbaldwin
ms.date: 07/03/2017
ms.openlocfilehash: 13470e85e4fe2741a11fcade3b97d333eb03efb7
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46465891"
---
# <a name="azure-key-vault-security-worlds-and-geographic-boundaries"></a>Azure Key Vault 安全世界和地理界限

Azure Key Vault 是多租用戶服務，並且在每個 Azure 位置使用硬體安全模組 (HSM) 的集區。 

同一個地理區域之 Azure 位置上的所有 HSM 都共用相同的密碼編譯界限 (Thales 安全世界)。 例如，美國東部和美國西部共用相同的安全世界，因為它們隸屬於美國地理位置。 同樣地，日本的所有 Azure 位置共用相同的安全世界，而位於澳洲、印度等國的所有 Azure 位置以此類推。 

## <a name="backup-and-restore-behavior"></a>備份與還原行為

在一個 Azure 位置的 Key Vault 建立的金鑰備份可還原至另一個 Azure 位置的 Key Vault，前提是這兩個條件都成立︰

- 兩個 Azure 位置都隸屬於相同的地理位置
- 兩個 Key Vault 都隸屬於相同的 Azure 訂用帳戶

例如，以指定的訂用帳戶在印度西部的 Key Vault 建立的金鑰備份，只能還原至同一個訂用帳戶和地理位置 (印度西部、印度中部或印度南部) 的另一個 Key Vault。

## <a name="regions-and-products"></a>區域和產品

- [Azure 區域](https://azure.microsoft.com/regions/)
- [依區域的 Microsoft 產品](https://azure.microsoft.com/regions/services/)

區域會對應至安全世界，在資料表中顯示為主要標題：

在依區域的產品文章中，例如，[美國] 索引標籤包含都會對應至美國區域的美國東部、美國中部、美國西部。 

>[!NOTE]
>例外狀況是美國 DOD 東部和美國 DOD 中部有自己的安全世界。 

同樣地，在 [歐洲] 索引標籤上，北歐和西歐都會對應到歐洲區域。 在 [亞太地區] 索引標籤上也是如此。



