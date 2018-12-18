---
title: Azure Marketplace 的計費選項 | Azure
description: Azure Marketplace 的發行者計費選項。
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
documentationcenter: ''
author: jm-aditi-ms
manager: pabutler
editor: ''
ms.assetid: ''
ms.service: marketplace
ms.workload: ''
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 06/05/2018
ms.author: ellacroi
ms.openlocfilehash: 67806563dcd67876d4d922c7f2c76379f7ad7222
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125508"
---
# <a name="billing-options-in-the-azure-marketplace"></a>Azure Marketplace 的計費選項

本文描述 [Azure Marketplace](https://azuremarketplace.microsoft.com) 可用的計費選項。

## <a name="commercial-considerations-in-the-marketplace"></a>Marketplace 相關的商業考量
Marketplace 不分享下列清單類型的營收： 
*   列出
*   試用版
*   使用「自備授權 (BYOL)」計費模型的交易

參與 Marketplace 店面並不會向您收取額外費用。

如需詳細資訊，請參閱 [Microsoft Azure Marketplace 參與政策](https://azure.microsoft.com/support/legal/marketplace/participation-policies)。  

## <a name="pay-as-you-go-and-byol-billing-options"></a>預付型方案和 BYOL 計費選項
使用「預付型方案」計費模型作為發行選項時，依使用量計費的軟體授權營收會由您與 Microsoft 依 80%/20% 的比例分成。 您可以同時使用「預付型方案」和 BYOL 計費模型，為單一供應項目定價。 這兩種計費模型在供應項目層級以不同的 SKU 共存。 您可以在「Cloud Partner 入口網站」中，於您的供應項目內設定計費模型。 

請思考一下以下範例：
*   如果您啟用 [預付型方案] 選項，就會有以下結果：
    | 授權成本 | 每小時美金 $1.00 元 |
    |:--- |:--- |
    | Azure 使用量成本 (D1/1 核心) | 每小時美金 $0.14 元 |
    | *由 Microsoft 向客戶收取的費用* | *每小時美金 $1.14 元* |

    在此情況下，使用您發佈的 VM 映像時，Microsoft 會收取每小時 $1.14 美元的費用：
    | Microsoft 收取的費用 | 每小時美金 $1.14 元 |
    |:--- |:--- |
    | Microsoft 向您支付授權成本的 80% | 每小時美金 $0.80 元 |
    | Microsoft 保留授權成本的 20% | 每小時美金 $0.20 元 |
    | Microsoft 保留 Azure 使用量成本 | 每小時美金 $0.14 元 |

*   如果您啟用 [BYOL] 選項，則會有以下結果：
    | 授權成本 | 由您協商和收取的授權費用 |
    |:--- |:--- |
    | Azure 使用量成本 (D1/1 核心) | 每小時美金 $0.14 元 |
    | *由 Microsoft 向客戶收取的費用* | *每小時美金 $0.14 元* |

    在此情況下，使用您發佈的 VM 映像時，Microsoft 會收取每小時 $0.14 美元的費用： 
    | Microsoft 收取的費用 | 每小時美金 $0.14 元 |
    |:--- |:--- |
    | Microsoft 保留 Azure 使用量成本 | 每小時美金 $0.14 元 |
    | Microsoft 保留授權成本的 0% | 每小時美金 $0.00 元 |

## <a name="single-billing-and-payment-methods"></a>單一計費和付款方式
使用 Marketplace 的「交易」發行選項有一個重要優點，就是會以單一計費方式直接向您的客戶收取授權費用和 Azure 使用量費用。

在此情況下，Microsoft 會代表您計費及收費。 Microsoft 計費服務可讓您無須建立自己與客戶的採購關係。 單一計費可節省您的時間和資源。 這也可以幫助您將精力放在銷售上，不用擔心帳款的催收。 

## <a name="enterprise-agreement"></a>Enterprise 合約  
如果您是 Microsoft Enterprise 合約客戶，可以使用 Enterprise 合約來支付 Microsoft 產品的款項。 可能會針對產品 (包括您的 Azure 使用量) 向您收取費用。 使用 Enterprise 合約作為付款方式適用於希望軟體和雲端服務的授權年限至少為三年的組織。 您可以分次付款，不用預付款項。 如果您使用「預付型方案」發行選項，則您的軟體授權費用會依每季的 Enterprise 合約超額計費週期計費。  

### <a name="monetary-commitment"></a>承諾用量
如果您是 Enterprise 合約客戶，您可以將 Azure 新增至您的合約。 向 Azure 事先承諾用量金額後，即可將 Azure 新增至您的合約。 您的承諾用量金額可供整年使用。 您的承諾用量包括 Azure 服務的任意使用組合。

## <a name="next-steps"></a>後續步驟
檢閱 [Azure Marketplace 和 AppSource 發行指南](./marketplace-publishers-guide.md)。
