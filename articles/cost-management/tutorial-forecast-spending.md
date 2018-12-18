---
title: 教學課程 - 在 Azure 中使用 Cloudyn 預測費用 | Microsoft Docs
description: 在本教學課程中，使用歷程記錄的使用方式及花費資料來預測花費。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/18/2018
ms.topic: tutorial
ms.service: cost-management
ms.custom: ''
manager: dougeby
ms.openlocfilehash: e1954ae9fcbb010c66e8a812e3e34477ac2bfc29
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49395219"
---
# <a name="tutorial-forecast-future-spending"></a>教學課程：預測未來的花費

Cloudyn 會使用歷程記錄的使用方式及費用資料，協助您預測未來的費用。 您可以使用 Cloudyn 報告來檢視所有成本預估的資料。 本教學課程中的範例會引導您使用報告來檢閱成本預估。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 預測未來的花費

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>必要條件

- 您必須具有 Azure 帳號。
- 您必須具有 Cloudyn 的試用版註冊或付費訂用帳戶。

## <a name="forecast-future-spending"></a>預測未來的花費

Cloudyn 包含成本預估報告，可協助您以一段時間的使用量作為基礎來預測花費。 其主要目的是協助您確保成本趨勢不會超過組織的期望。 您使用的報告是「當月預估成本」和「年度預估成本」。 如果您的使用方式維持在與過去 30 天內的使用方式一致，兩個報告就都會顯示預估的未來花費。

「當月預估成本」報告會顯示您服務的成本。 它會使用月份開頭和前一個月的成本來顯示預估的成本。 在入口網站頂端的 [報告] 功能表上，按一下 [成本] > [預估和預算] > [當月預估成本]。 下圖說明一個範例。

![當月預估成本](./media/tutorial-forecast-spending/project-month01.png)

在範例中，您可以查看哪些服務的花費最大。 Azure 的成本低於 AWS 成本。 如果您需要查看 Azure VM 的成本預估詳細資料，請在 [篩選] 清單中選取 [Azure/VM]。

![Azure VM 當月預估成本](./media/tutorial-forecast-spending/project-month02.png)

請遵循相同的基本上述步驟，看看您感興趣的其他服務每月成本預估。

「年度預估成本」報告會顯示您的服務接下來 12 個月的推斷成本。

在入口網站頂端的 [報告] 功能表上，按一下 [成本] > [預估和預算] > [年度預估成本]。 下圖說明一個範例。

![年度預估成本報告](./media/tutorial-forecast-spending/project-annual01.png)

在範例中，您可以查看哪些服務的花費最大。 如同每月範例，Azure 成本低於 AWS 成本。 如果您需要查看 Azure VM 的成本預估詳細資料，請在 [篩選] 清單中選取 [Azure/VM]。

![VM 的年度預估成本](./media/tutorial-forecast-spending/project-annual02.png)

在上圖中，Azure VM 的年度預估成本為 $28,374。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 預測未來的花費


請移至下一個教學課程，以了解透過成本配置和回報報表來管理成本的方式。

> [!div class="nextstepaction"]
> [以成本配置和回報報表來管理成本](tutorial-manage-costs.md)
