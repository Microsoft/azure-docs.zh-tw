---
title: 監視和追蹤 Azure 免費服務的使用量 | Microsoft Docs
description: 了解如何檢查免費服務的使用量 使用 Azure 入口網站和使用量 csv。
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
ms.author: cwatson
ms.openlocfilehash: df049a87763f3aae8da2db153f876b88ed39b988
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47390233"
---
# <a name="check-usage-of-free-services-included-with-your-azure-free-account"></a>檢查 Azure 免費帳戶隨附的免費服務使用量 

對於 Azure 免費帳戶隨附的服務，除非您超過這些服務的限額，否則不會向您收費。 為了保持在限額內，您可以使用 Azure 入口網站或使用量檔案來監視和追蹤免費服務的使用量。 

## <a name="check-usage-on-the-azure-portal"></a>在 Azure 入口網站上檢查使用量

1.  登入 [Azure 入口網站]( http://portal.azure.com)。

2.  在左側導覽區域中，選取 [所有服務]。

3.  選取 **訂用帳戶** 。

4.  選取您註冊免費帳戶時建立的訂用帳戶。

    ![顯示全部訂用帳戶的螢幕擷取畫面](./media/billing-check-usage-of-free-services/select-free-account-subscription.png)

5.  概觀區段會顯示與您的訂用帳戶有關的基本資訊，例如訂用帳戶識別碼、供應項目類型和訂用帳戶名稱。 您也可以找到免費帳戶信用額度到期的資訊。

    ![顯示訂用帳戶基本資訊的螢幕擷取畫面](./media/billing-check-usage-of-free-services/subscription-essential-information.png)

6.  向下捲動找出目前及預估費用的資訊。 費用包括免費帳戶未隨附的服務使用量，以及超過免費服務限額的使用量。 

    ![顯示訂用帳戶費用資訊的螢幕擷取畫面](./media/billing-check-usage-of-free-services/subscription-cost-information.png)

7.  概觀區段的最後一個部分有免費服務使用量的表格。 

    ![顯示免費服務使用量的螢幕擷取畫面](./media/billing-check-usage-of-free-services/subscription-usage-free-services.png)

    該表格包含下列資料行：

* **計量名稱：** 識別耗用計量的量值單位。 若要了解服務與計量的對應，請參閱[了解免費服務與計量的對應](billing-understand-free-service-meter-mapping.md)。 
* **使用量/限制：** 目前月份的使用量和計量的限額。 您也可在狀態列中找到此資訊。
* **狀態：** 計量的使用量狀態。 根據您的使用量模式而定，會出現下列其中一個狀態。
  * **不在使用中：** 您未使用計量，或計費的使用量未達到計費系統的標準。
  * **已在 \<Date> 超過：** 您在 \<Date> 超過計量的限額。
  * **不太可能超過：** 您不太可能超過計量的限額。
  * **將在 \<Date> 超過：** 您可能在 \<Date> 超過計量的限額。


## <a name="check-usage-through-the-usage-file"></a>透過使用量檔案查看使用量

使用量檔案提供更細部的 Azure 訂用帳戶資訊。 您可以從 Azure 帳戶中心下載每月和每日使用量檔案。 若要了解如何下載使用量檔案，並了解必要的存取，請參閱[取得發票和使用量](billing-download-azure-invoice-daily-usage-date.md)。 若要了解使用量檔案中的資料行，請參閱[了解使用量的字詞](billing-understand-your-usage.md)。 

使用量檔案包含免費和付費服務的使用量資訊。 免費服務計量的計量名稱結果會附加**免費**。 若要尋找免費計量，請在 Excel 中開啟該檔案，並篩選**計量類別資料行**找出包含文字的儲存格 **- 免費** (使用文字篩選條件&rarr;包含篩選條件&nbsp;)

![顯示免費服務使用量的螢幕擷取畫面](./media/billing-check-usage-of-free-services/free-services-usage-csv.png)


## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
