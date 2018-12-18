---
title: 針對 Azure Web Apps 購買自訂網域名稱
description: 了解如何在 Azure App Service 中購買搭配 Web 應用程式的自訂網域名稱。
services: app-service\web
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: 70fb0e6e-8727-4cca-ba82-98a4d21586ff
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/24/2017
ms.author: cephalin
ms.openlocfilehash: 48e0e68794e83739835d97aa8a2b26516c660357
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39345186"
---
# <a name="buy-a-custom-domain-name-for-azure-web-apps"></a>針對 Azure Web Apps 購買自訂網域名稱

App Service 網域 (預覽) 是直接在 Azure 中管理的頂層網域。 它們可以讓 [Azure Web Apps](app-service-web-overview.md) 的自訂網域管理作業變得很簡單。 本教學課程會示範如何購買 App Service 網域，並將 DNS 名稱指派給 Azure Web Apps。

本文適用於 Azure App Service (Web Apps、API Apps、Mobile Apps、Logic Apps)。 若為 Azure VM 或 Azure 儲存體，請參閱[將 App Service 網域指派給 Azure VM 或 Azure 儲存體](https://blogs.msdn.microsoft.com/appserviceteam/2017/07/31/assign-app-service-domain-to-azure-vm-or-azure-storage/)。 若為雲端服務，請參閱[設定 Azure 雲端服務的自訂網域名稱](../cloud-services/cloud-services-custom-domain-name-portal.md)。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

* [建立 App Service 應用程式](/azure/app-service/)，或使用您針對另一個教學課程建立的應用程式。
* [移除訂用帳戶的消費限制](../billing/billing-spending-limit.md#remove)。 您無法使用免費的訂用帳戶信用額度購買 App Service 網域。

## <a name="prepare-the-app"></a>準備應用程式

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

若要在 Azure Web Apps 中使用自訂網域，Web 應用程式的 [App Service 方案](https://azure.microsoft.com/pricing/details/app-service/)必須是付費層 (「共用」、「基本」、「標準」或「進階」)。 在此步驟中，您要確定 Web 應用程式位於支援的定價層。

### <a name="sign-in-to-azure"></a>登入 Azure

開啟 [Azure 入口網站](https://portal.azure.com)並使用您的 Azure 帳戶登入。

### <a name="navigate-to-the-app-in-the-azure-portal"></a>瀏覽至 Azure 入口網站中的應用程式

從左側功能表，選取 [App Service]，然後選取應用程式的名稱。

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/select-app.png)

您看到 App Service 應用程式的管理分頁。  

### <a name="check-the-pricing-tier"></a>檢查定價層

在應用程式分頁的左側導覽中，捲動到 [設定] 區段，然後選取 [相應增加 (App Service 方案)]。

![相應增加功能表](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

會以藍色框線醒目顯示應用程式目前的層。 請檢查以確定您的應用程式不是位於 **F1** 層。 **F1** 層不支援自訂 DNS。 

![檢查定價層](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

如果 App Service 方案不是位於 **F1** 層，請關閉 [相應增加] 頁面，然後跳至 [[購買網域]](#buy-the-domain)。

### <a name="scale-up-the-app-service-plan"></a>相應增加 App Service 方案

選取任何非免費層 (**D1**、**B1**、**B2**、**B3** 或「生產」類別中的任何一層)。 如需其他選項，請按一下 [查看其他選項]。

按一下 [套用]。

![檢查定價層](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

當您看見下列通知時，表示擴充作業已完成。

![擴充作業確認](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

## <a name="buy-the-domain"></a>購買網域

### <a name="pricing-information"></a>定價資訊
如需 Azure App Service 網域的定價資訊，請瀏覽 [App Service 定價頁面](https://azure.microsoft.com/pricing/details/app-service/windows/)並向下捲動至＜App Service 網域＞。

### <a name="sign-in-to-azure"></a>登入 Azure
開啟 [Azure 入口網站](https://portal.azure.com/)並使用您的 Azure 帳戶登入。

### <a name="launch-buy-domains"></a>啟動購買網域
在 [Web Apps] 索引標籤中，按一下您 Web 應用程式的名稱，並選取 [設定]，然後選取 [自訂網域]
   
![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

在 [自訂網域] 頁面中，按一下 [購買網域]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-1.png)

> [!NOTE]
> 如果您沒看見 [App Service 網域] 區段，則需要移除您 Azure 帳戶的消費限制 (請參閱[必要條件](#prerequisites))。
>
>

### <a name="configure-the-domain-purchase"></a>設定網域購買

在 [App Service 網域] 頁面的 [搜尋網域] 方塊中，輸入要購買的網域名稱，然後輸入 `Enter`。 建議的可用網域會顯示在文字方塊下方。 選取一或多個要購買的網域。
   
![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-2.png)

> [!NOTE]
> App Service 網域支援下列[頂層網域](https://wikipedia.org/wiki/Top-level_domain)：_com_、_net_、_co.uk_、_org_、_nl_、_in_、_biz_、_org.uk_，以及 _co.in_。
>
>

按一下 [連絡人資訊]，然後填寫網域的連絡人資訊表單。 完成之後，請按一下 [確定] 以返回 [App Service 網域] 頁面。
   
請務必填寫所有必要欄位，並盡可能正確填寫。 連絡資訊的資料若不正確，可能會導致無法購買網域。 

接下來，請為網域選取所需選項。 請參閱下表中的說明：

| 設定 | 建議的值 | 說明 |
|-|-|-|
|隱私權保護 | 啟用 | 選擇加入「隱私權保護」，此服務已_免費_包含在購買價格中。 有些頂層網域受控於不支援隱私權保護的註冊機構，會將之列在 [隱私權保護] 頁面中。 |
| 指派預設主機名稱 | **www** 和 **@** | 如有需要，請選取所需的主機名稱繫結。 網域購買作業完成時，Web 應用程式就可以從選取的主機名稱存取。 如果 Web 應用程式受 [Azure 流量管理員](https://azure.microsoft.com/services/traffic-manager/)管理，您就不會看到指派根網域 (@) 的選項，因為流量管理員不支援 A 記錄。 您可以在網域購買完成之後變更主機名稱指派。 |

### <a name="accept-terms-and-purchase"></a>接受條款並購買

按一下 [法律條款] 來檢閱條款與費用，然後按一下 [購買]。

> [!NOTE]
> App Service 網域使用 Azure DNS 來裝載網域。 除了網域註冊費之外，您也必須支付 Azure DNS 的使用費用。 如需相關資訊，請參閱 [Azure DNS 定價](https://azure.microsoft.com/pricing/details/dns/)。
>
>

回到 [App Service 網域] 頁面，然後按一下 [確定]。 作業進行中時，您會看到下列通知：

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-validate.png)

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-purchase-success.png)

### <a name="test-the-hostnames"></a>測試主機名稱

如果您已經指派預設主機名稱給 Web 應用程式，也會看見針對每個選取主機名稱的成功通知。 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-bind-success.png)

您也會在 [自訂網域] 頁面的 [自訂主機名稱] 區段中看到選取的主機名稱。 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostnames-added.png)

若要測試主機名稱，請在瀏覽器中瀏覽至列出的主機名稱。 在上方螢幕擷取畫面的範例中，嘗試瀏覽至 _kontoso.net_ 和 _www.kontoso.net_。

## <a name="assign-hostnames-to-web-app"></a>將主機名稱指派給 Web 應用程式

如果您在購買程序期間選擇不指派一或多個預設主機名稱給 Web 應用程式，或是需要指派未列出的主機名稱，您隨時都可以指派主機名稱。

您也可以將 App Service 網域中的主機名稱指派給任何其他 Web 應用程式。 其步驟需視 App Service 網域和 Web 應用程式是否屬於同一訂用帳戶而定。

- 不同的訂用帳戶：以和對應外部購買的網域相同的方式，將來自 App Service 網域的自訂 DNS 記錄對應至 Web 應用程式。 如需將自訂 DNS 名稱新增至 App Service 網域的相關資訊，請參閱[管理自訂 DNS 記錄](#custom)。 若要將外部購買的網域對應至 Web 應用程式，請參閱[將現有的自訂 DNS 名稱對應至 Azure Web Apps](app-service-web-tutorial-custom-domain.md)。 
- 相同的訂用帳戶：請使用下列步驟。

### <a name="launch-add-hostname"></a>啟動新增主機名稱
在 [應用程式服務] 頁面中，選取要指派主機名稱的 Web 應用程式名稱，選取 [設定]，然後選取 [自訂網域]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

請確定您所購買的網域已列於 [App Service 網域] 區段中，但請不要選取它。 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-select-domain.png)

> [!NOTE]
> 相同訂用帳戶中的所有 App Service 網域，都會顯示在 Web 應用程式的 [自訂網域] 頁面中。 如果您的網域已位於 Web 應用程式的訂用帳戶中，但您在 Web 應用程式的 [自訂網域] 頁面中看不到該網域，請嘗試重新開啟 [自訂網域] 頁面，或是重新整理該網頁。 此外，請透過檢查位於 Azure 入口網站頂端的通知鈴，來取得進度或建立失敗的訊息。
>
>

選取 [新增主機名稱]。

### <a name="configure-hostname"></a>設定主機名稱
在 [新增主機名稱] 對話方塊中，輸入 App Service 網域或任何子網域的完整網域名稱。 例如︰

- kontoso.net
- www.kontoso.net
- abc.kontoso.net

完成之後，請選取 [驗證]。 系統會自動為您選取主機名稱記錄類型。

選取 [新增主機名稱]。

作業完成之後，您會看到成功指派主機名稱的通知。  

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-bind-success.png)

### <a name="close-add-hostname"></a>關閉新增主機名稱
在 [新增主機名稱] 頁面中，視需求將任何其他主機名稱指派給您的 Web 應用程式。 完成之後，請關閉 [新增主機名稱] 頁面。

您現在應該會在應用程式的 [自訂網域] 頁面中看到新指派的主機名稱。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostnames-added2.png)

### <a name="test-the-hostnames"></a>測試主機名稱

請在瀏覽器中瀏覽至列出的主機名稱。 在上方螢幕擷取畫面的範例中，嘗試瀏覽至 _abc.kontoso.net_。

## <a name="renew-the-domain"></a>續訂網域

您購買的 App Service 網域自購買起一年內有效。 按照預設，網域會設定為自動續訂下一年度，並使用您的付款方式付費。 如果您想要關閉自動續訂，或想要以手動方式續訂網域，請依照下列步驟進行。

在 [Web Apps] 索引標籤中，按一下您 Web 應用程式的名稱，選取 [設定]，然後選取 [自訂網域]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

在 [App Service 網域] 區段中，選取您想要設定的網域。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-select-domain.png)

從網域的左側導覽中，選取 [網域續訂]。 若要停止自動續訂您的網域，請選取 [關閉]，然後選取 [儲存]。 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-autorenew.png)

若要以手動方式續訂您的網域，請選取 [續訂網域]。 不過，此按鈕在網域到期前 90 天才會啟用。

<a name="custom"></a>

## <a name="manage-custom-dns-records"></a>管理自訂 DNS 記錄

在 Azure 中，App Service 網域的 DNS 記錄是使用 [Azure DNS](https://azure.microsoft.com/services/dns/) 來管理。 和針對外部購買的網域一樣，您可以新增、移除及更新 DNS 記錄。

### <a name="open-app-service-domain"></a>開啟 App Service 網域

從 Azure 入口網站中的左側功能表，選取 [所有服務] > [App Service 網域]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-access.png)

選取要管理的網域。 

### <a name="access-dns-zone"></a>存取 DNS 區域

在網域的左側功能表中，選取 [DNS 區域]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-dns-zone.png)

這個動作會開啟您在 Azure DNS 中之 App Service 網域的 [DNS 區域](../dns/dns-zones-records.md)頁面。 如需如何編輯 DNS 記錄的相關資訊，請參閱[如何在 Azure 入口網站中管理 DNS 區域](../dns/dns-operations-dnszones-portal.md)。

## <a name="cancel-purchase-delete-domain"></a>取消購買 (刪除網域)

在您購買 App Service 網域之後，可以在五天內取消購買並獲得全額退費。 您也可以在五天之後刪除 App Service 網域，但無法收到退款。

### <a name="open-app-service-domain"></a>開啟 App Service 網域

從 Azure 入口網站中的左側功能表，選取 [所有服務] > [App Service 網域]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-access.png)

選取您要取消或刪除的網域。 

### <a name="delete-hostname-bindings"></a>刪除主機名稱繫結

在網域的左側功能表中，選取 [主機名稱繫結]。 這裡會列出來自所有 Azure 服務的主機名稱繫結。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostname-bindings.png)

您必須先刪除所有主機名稱繫結，才能刪除 App Service 網域。

請透過選取 **...** > [刪除].來刪除每個主機名稱繫結。 刪除所有繫結之後，請選取 [儲存]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-delete-hostname-bindings.png)

### <a name="cancel-or-delete"></a>取消或刪除

在網域的左側功能表中，選取 [概觀]。 

如果已購買網域的取消期間尚未到期，請選取 [取消購買]。 否則，您會看到 [刪除] 按鈕。 若要在不退款情況下刪除網域，請選取 [刪除]。

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-cancel.png)

若要確認作業，請選取 [是]。

作業完成之後，該網域就會從您的訂用帳戶中釋放，並可再次供任何人購買。 

## <a name="direct-default-url-to-a-custom-directory"></a>將預設 URL 導向自訂目錄

根據預設，App Service 會將 Web 要求導向應用程式程式碼的根目錄。 若要將這些要求導向子目錄 (例如 `public`)，請參閱[將預設 URL 導向自訂目錄](app-service-web-tutorial-custom-domain.md#virtualdir)。

## <a name="more-resources"></a>其他資源

[常見問題集：App Service 網域 (預覽) 與自訂網域](https://blogs.msdn.microsoft.com/appserviceteam/2017/08/08/faq-app-service-domain-preview-and-custom-domains/)
