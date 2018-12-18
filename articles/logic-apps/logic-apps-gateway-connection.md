---
title: 存取 Azure Logic Apps 的內部部署資料來源 | Microsoft Docs
description: 建立及設定內部部署資料閘道，以便從邏輯應用程式存取內部部署資料來源
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: article
ms.date: 07/20/2018
ms.reviewer: yshoukry, LADocs
ms.suite: integration
ms.openlocfilehash: 65c7e03b349314ad61fa5f1ea8322f4d1352b8e6
ms.sourcegitcommit: 727a0d5b3301fe20f20b7de698e5225633191b06
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39145684"
---
# <a name="connect-to-data-sources-on-premises-from-azure-logic-apps-with-on-premises-data-gateway"></a>透過內部部署資料閘道從 Azure Logic Apps 連線至內部部署資料來源

若要從邏輯應用程式存取內部部署資料來源，您可以在 Azure 中建立資料閘道資源，以便邏輯應用程式使用[內部部署連接器](../logic-apps/logic-apps-gateway-install.md#supported-connections)。 本文示範如何在[本機電腦下載並安裝閘道](../logic-apps/logic-apps-gateway-install.md)之後，建立 Azure 閘道資源。 

如需如何使用閘道與其他服務的資訊，請參閱下列文章：

* [Microsoft Power BI 內部部署資料閘道](https://powerbi.microsoft.com/documentation/powerbi-gateway-onprem/)
* [Microsoft Flow 內部部署資料閘道](https://flow.microsoft.com/documentation/gateway-manage/)
* [Microsoft PowerApps 內部部署資料閘道](https://powerapps.microsoft.com/tutorials/gateway-management/)
* [Azure Analysis Services 內部部署資料閘道](../analysis-services/analysis-services-gateway.md)

## <a name="prerequisites"></a>必要條件

* 您已經[在本機電腦下載並安裝資料閘道](../logic-apps/logic-apps-gateway-install.md)。

* 您的閘道安裝尚未與 Azure 中的閘道資源相關聯。 您可以將閘道安裝只連結至一個閘道資源，當建立閘道資源並選取閘道安裝時會發生這種情況。 此連結讓其他資源無法使用閘道安裝。

* 當您登入 Azure 入口網站並建立閘道資源時，您必須使用先前用來[安裝內部部署資料閘道](../logic-apps/logic-apps-gateway-install.md#requirements)的同一個登入帳戶。
您也必須使用在安裝閘道時所用的同一個 [Azure 訂用帳戶](https://docs.microsoft.com/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer)。 如果您還沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。

* 若要在 Azure 入口網站中建立和維護閘道資源，[Windows 服務帳戶](../logic-apps/logic-apps-gateway-install.md#windows-service-account)必須至少具備**參與者**權限。 內部部署資料閘道會以 Windows 服務身分執行，並且設定成使用 `NT SERVICE\PBIEgwService` 作為 Windows 服務登入認證。 

  > [!NOTE]
  > Windows 服務帳戶與用來連線至內部部署資料來源的帳戶不同，也與用來登入雲端服務的公司或學校 Azure 帳戶不同。

## <a name="download-and-install-gateway"></a>下載並安裝閘道

您必須已在本機電腦上安裝閘道，才可以繼續進行本文中的步驟。
如果您還沒有這麼做，請依照步驟來[下載並安裝內部部署資料閘道](../logic-apps/logic-apps-gateway-install.md)。 

<a name="create-gateway-resource"></a>

## <a name="create-azure-resource-for-gateway"></a>為閘道建立 Azure 資源

在本機電腦上安裝閘道之後，您可接著為閘道建立 Azure 資源。 此步驟也會讓閘道資源與 Azure 訂用帳戶產生關聯。

1. 登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。 請務必使用在安裝閘道時所用的同一個 Azure 公司或學校電子郵件地址。

2. 在主要 Azure 功能表上，選取 [建立資源] > 
[整合] > [內部部署資料閘道]。

   ![尋找 [內部部署資料閘道]](./media/logic-apps-gateway-connection/find-on-premises-data-gateway.png)

3. 在 [建立連線閘道] 頁面上，為閘道資源提供這項資訊：

   | 屬性 | 說明 | 
   |----------|-------------|
   | **名稱** | 您的閘道資源名稱 | 
   | **訂用帳戶** | 您的 Azure 訂用帳戶名稱，該名稱必須是與邏輯應用程式相同的訂用帳戶。 預設的訂用帳戶會由您用來登入的 Azure 帳戶決定。 | 
   | **資源群組** | 可用來組織相關資源的 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)名稱 | 
   | **位置** | Azure 對此位置有所限制，它的所在區域必須和[閘道安裝](../logic-apps/logic-apps-gateway-install.md)期間為閘道雲端服務所選取的區域相同。 <p>**注意**：請確定此閘道資源位置符合閘道雲端服務的位置。 否則，已安裝的閘道清單中可能不會出現您的閘道安裝，從而供您在下一個步驟中選取。 閘道資源和邏輯應用程式可使用不同的區域。 | 
   | **安裝名稱** | 如果您的閘道安裝還不是選取狀態，請選取您先前安裝的閘道。 | 
   | | | 

   下列是一個範例：

   ![提供詳細資料以建立內部部署資料閘道](./media/logic-apps-gateway-connection/createblade.png)

4. 若要在 Azure 儀表板中新增閘道資源，請選取 [釘選到儀表板]。 完成之後，請選擇 [建立]。

   您可以從主要 Azure 功能表中選取 [所有服務]，隨時尋找或檢視閘道。 
   在搜尋方塊中，輸入「內部部署資料閘道」，然後選取 [內部部署資料閘道]。

   ![尋找 [內部部署資料閘道]](./media/logic-apps-gateway-connection/find-on-premises-data-gateway-enterprise-integration.png)

<a name="connect-logic-app-gateway"></a>

## <a name="connect-to-on-premises-data"></a>連線至內部部署資料

在您建立閘道資源並且讓 Azure 訂用帳戶與此資源相關聯之後，即可使用閘道，建立邏輯應用程式與內部部署資料來源之間的連線。

1. 在 Azure 入口網站中，於邏輯應用程式設計工具中建立或開啟邏輯應用程式。

2. 新增可支援內部部署連線的連接器，例如 **SQL Server**。

3. 現在設定您的連線：

   1. 選取 [透過內部部署資料閘道連線]。 

   2. 針對 [閘道]，選取您先前建立的閘道資源。 

      雖然閘道連線位置必須與邏輯應用程式位於相同區域，但您可以選取不同區域中的閘道。

   3. 提供唯一連線名稱和其他必要資訊。 

      唯一的連線名稱可在之後協助您輕鬆識別該連線，尤其是當您建立了多個連線時。 如果情況允許，也請在使用者名稱中包含完整網域。
   
      下列是一個範例：

      ![建立邏輯應用程式與資料閘道之間的連線](./media/logic-apps-gateway-connection/blankconnection.png)

   4. 完成之後，請選擇 [建立]。 

您的閘道連線現已可供邏輯應用程式使用。

## <a name="edit-connection"></a>編輯連線

在為邏輯應用程式建立閘道連線之後，您之後或許會想更新該特定連線的設定。

1. 尋找您的閘道連線：

   * 若只要尋找邏輯應用程式的所有 API 連線，請在邏輯應用程式功能表的 [開發工具] 底下，選取 [API 連線]。 
   
     ![移至邏輯應用程式，選取 [API 連線]](./media/logic-apps-gateway-connection/logic-app-find-api-connections.png)

   * 若要尋找與您的 Azure 訂用帳戶相關聯的所有 API 連線： 

     * 從主要 Azure 功能表，移至 [所有資源] > [Web] > [API 連線]。 
     * 或者，從主要 Azure 功能表，移至 [所有資源]。

2. 選取您想要的閘道連線，然後選擇 [編輯 API 連線]。

   > [!TIP]
   > 如果您的更新未生效，請嘗試[先停止再重新啟動閘道 Windows 服務](./logic-apps-gateway-install.md#restart-gateway)。

<a name="change-delete-gateway-resource"></a>

## <a name="delete-gateway-resource"></a>刪除閘道資源

若要建立不同的閘道資源、讓閘道與其他資源相關連，或是要移除閘道資源，您可以直接刪除閘道資源，這並不會影響閘道安裝。 

1. 從主要 Azure 功能表，移至 [所有資源]。 

2. 尋找並選取您的閘道資源。

3. 如果尚未選取，請在您的閘道資源功能表上選取 [內部部署資料閘道]。 

4. 在資源工具列上，選擇 [刪除]。

<a name="faq"></a>

## <a name="frequently-asked-questions"></a>常見問題集

[!INCLUDE [existing-gateway-location-changed](../../includes/logic-apps-existing-gateway-location-changed.md)]

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* [保護邏輯應用程式](./logic-apps-securing-a-logic-app.md)
* [Logic Apps 範例和常見案例](./logic-apps-examples-and-scenarios.md)
