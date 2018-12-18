---
title: 取消 Azure 訂用帳戶 | Microsoft Docs
description: 說明如何取消 Azure 訂用帳戶，例如免費試用訂用帳戶
services: ''
documentationcenter: ''
author: genlin
manager: jlian
editor: ''
tags: billing
ms.assetid: 3051d6b0-179f-4e3a-bda4-3fee7135eac5
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: genli
ms.openlocfilehash: 0bd0fca54ae5bf4292564a15adf38b7586768450
ms.sourcegitcommit: 727a0d5b3301fe20f20b7de698e5225633191b06
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39145446"
---
# <a name="cancel-your-subscription-for-azure"></a>取消您的 Azure 訂用帳戶

您可以用[帳戶管理員](billing-subscription-transfer.md#whoisaa)身分取消您的 Azure 訂用帳戶。 取消訂用帳戶之後，您將無法存取 Azure 服務和資源端點。

取消訂用帳戶之前︰

* 備份您的資料。 例如，如果您將資料儲存在 Azure 儲存體或 SQL，請下載複本。 如果您有虛擬機器，請將其映像儲存在本機。
* 關閉您的服務。 移至[管理入口網站中的資源頁面](https://ms.portal.azure.com/?flight=1#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fresources)，並**停止**任何執行中的虛擬機器、應用程式或其他服務。
* 考慮移轉您的資料。 請參閱[將資源移動到新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md)。

如果您取消付費 Azure 支援方案，剩餘的訂閱期間仍會計費。 如需詳細資訊，請參閱 [Azure 支援方案](https://azure.microsoft.com/support/plans/)。

## <a name="cancel-subscription-using-the-azure-portal"></a>使用 Azure 入口網站取消訂用帳戶

1. 在 [Azure 入口網站的 [訂用帳戶] 頁面](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)中選取您的訂用帳戶。
1. 選取您要取消的訂用帳戶，然後按一下 [取消訂用帳戶]。

    ![顯示 [取消] 按鈕的螢幕擷取畫面](./media/billing-how-to-cancel-azure-subscription/cancel_ibiza.png)
1. 遵循提示並完成取消作業。

## <a name="what-happens-after-i-cancel-my-subscription"></a>取消訂用帳戶之後會發生什麼情況？

當您取消之後，計費會立即停止。 不過，可能需要最多 10 分鐘的時間，取消作業才會出現在入口網站中。

在這之後，您的服務將會停用。 這表示系統會解除配置您的虛擬機器、釋出暫時 IP 位址，且儲存體會變成唯讀。

如果您在計費期間中途取消訂用帳戶，我們會在該期間結束後的一般發票核發日傳送最後一張帳單。 

我們會在 90 天後才永久刪除您的資料，以防您需要存取該資料或改變心意。 我們不會向您收取保留資料的費用。 若要深入了解，請參閱 [Microsoft 信任中心 - 我們如何管理您的資料](https://go.microsoft.com/fwLink/p/?LinkID=822930&clcid=0x409) \(英文\)。

## <a name="reactivate-subscription"></a>重新啟動訂用帳戶

如果您不小心取消隨用隨付訂用帳戶，您可以[在帳戶中心重新啟動它](billing-subscription-become-disable.md)。

如果您的訂用帳戶不是隨用隨付類型，請在取消後 90 天內與支援服務連絡，以重新啟動您的訂用帳戶。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果您仍有問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
