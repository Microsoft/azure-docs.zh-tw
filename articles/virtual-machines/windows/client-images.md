---
title: 在 Azure 中使用 Windows 用戶端映像 | Microsoft Docs
description: 如何使用 Visual Studio 訂用帳戶權益，在 Azure 中部署 Windows 7、Windows 8 或 Windows 10，以進行開發/測試案例
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: 91c3880a-cede-44f1-ae25-f8f9f5b6eaa4
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: eb227b4b4cbdc40a4e708726bbe0f5d0c7eacb74
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49364660"
---
# <a name="use-windows-client-in-azure-for-devtest-scenarios"></a>在 Azure 中使用 Windows 用戶端進行開發/測試案例
假設您有適當的 Visual Studio (先前稱為 MSDN) 訂閱，您可以在 Azure 中使用 Windows 7、Windows 8 或 Windows 10 企業版(x64) 進行開發/測試案例。 這篇文章概述在 Azure 中執行 Windows 7、Windows 8.1、Windows 10 企業版和使用下列「Azure 資源庫」映像的資格需求。

![Azure 入口網站中的映像詳細資料](./media/client-images/windows-client-msdn-images.png) 

> [!NOTE]
> 針對 Azure 資源庫中的 Windows 10 專業版和 Windows 10 Pro N 映像，請參閱[如何使用多租用戶主機權限在 Azure 上部署 Windows 10](windows-desktop-multitenant-hosting-deployment.md)
>![Azure 入口網站中的映像詳細資料](./media/client-images/windows-client-pro-images.png) 
>

## <a name="subscription-eligibility"></a>訂用帳戶資格
有效的 Visual Studio 訂閱者 (已取得 Visual Studio 訂用帳戶授權的人) 可以使用 Windows 用戶端進行開發和測試。 Windows 用戶端可以用在您自己的硬體及以任何類型的 Azure 訂用帳戶執行的 Azure 虛擬機器。 Windows 用戶端無法部署至或用於 Azure 來從事一般實際執行用途，或由不是有效 Visual Studio 訂閱者的人使用。

為方便您使用，Azure 資源庫的[符合資格者的開發/測試供應項目](#eligible-offers)內提供特定的 Windows 10 映像。 屬於任一供應項目類型的 Visual Studio 訂閱者也可以[適當地準備和建立](prepare-for-upload-vhd-image.md) 64 位元的 Windows 7、Windows 8 或 Windows 10 映像，然後[上傳至 Azure](upload-generalized-managed.md)。 僅限有效的 Visual Studio 訂閱者用於開發/測試用途。

## <a name="eligible-offers"></a>符合資格者的供應項目
下表詳細說明可透過 Azure 資源庫部署 Windows 10 的供應項目識別碼。 只有在下列供應項目中才能看到 Windows 10 映像。 需以其他供應項目類型執行 Windows 用戶端的 Visual Studio 訂閱者則需要[適當地準備和建立](prepare-for-upload-vhd-image.md) 64 位元的 Windows 7、Windows 8 或 Windows 10 映像，然後[上傳至 Azure](upload-generalized-managed.md)。

| 供應項目名稱 | 供應項目號碼 | 可用的用戶端映像 |
|:--- |:---:|:---:|
| [隨用隨付開發/測試](https://azure.microsoft.com/offers/ms-azr-0023p/) |0023P |Windows 10 |
| [Visual Studio Enterprise (MPN) 訂閱者](https://azure.microsoft.com/offers/ms-azr-0029p/) |0029P |Windows 10 |
| [Visual Studio Professional 訂閱者](https://azure.microsoft.com/offers/ms-azr-0059p/) |0059P |Windows 10 |
| [Visual Studio Test Professional 訂閱者](https://azure.microsoft.com/offers/ms-azr-0060p/) |0060P |Windows 10 |
| [Visual Studio Premium 含 MSDN (權益)](https://azure.microsoft.com/offers/ms-azr-0061p/) |0061P |Windows 10 |
| [Visual Studio Enterprise 訂閱者](https://azure.microsoft.com/offers/ms-azr-0063p/) |0063P |Windows 10 |
| [Visual Studio Enterprise (BizSpark) 訂閱者](https://azure.microsoft.com/offers/ms-azr-0064p/) |0064P |Windows 10 |
| [Enterprise 開發/測試](https://azure.microsoft.com/offers/ms-azr-0148p/) |0148P |Windows 10 |

## <a name="check-your-azure-subscription"></a>檢查您的 Azure 訂用帳戶
如果您不知道您的供應項目識別碼，您可以透過 Azure 入口網站以下列兩種方式取得：  

- 在 [訂用帳戶] 視窗上︰

  ![Azure 入口網站中的供應項目識別碼詳細資料](./media/client-images/offer-id-azure-portal.png) 

- 或者，按一下 [計費]，然後按一下訂用帳戶識別碼。 供應項目識別碼隨即會出現在 [計費] 視窗中。

您也可以從 Azure 帳戶入口網站的 [[訂用帳戶] 索引標籤](http://account.windowsazure.com/Subscriptions) 檢視供應項目識別碼︰

![Azure 帳戶入口網站中的供應項目識別碼詳細資料](./media/client-images/offer-id-azure-account-portal.png) 

## <a name="next-steps"></a>後續步驟
您現在可以使用 [PowerShell](quick-create-powershell.md)、[Resource Manager templates](ps-template.md) 或 [Visual Studio](../../vs-azure-tools-resource-groups-deployment-projects-create-deploy.md) 來部署您的 VM。

