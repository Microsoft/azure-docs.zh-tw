---
title: 使用 HTTPS 端點管理潛在客戶 - Microsoft 商業市集
description: 了解如何使用 Power Automate 和 HTTPS 端點來管理 Microsoft AppSource 和 Azure Marketplace 中的潛在客戶。
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 03/30/2020
ms.openlocfilehash: 5bea2cf256e30bd896957bbee0e0ad824057a569
ms.sourcegitcommit: 08458f722d77b273fbb6b24a0a7476a5ac8b22e0
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2021
ms.locfileid: "98247177"
---
# <a name="use-an-https-endpoint-to-manage-commercial-marketplace-leads"></a>使用 HTTPS 端點來管理商業市集潛在客戶

如果您的客戶關係管理 (CRM) 系統在合作夥伴中心中尚未明確支援接收 Microsoft AppSource 和 Azure Marketplace 潛在客戶，您可以使用 [Power Automate](https://powerapps.microsoft.com/automate-processes/) 中的 HTTPS 端點來處理這些潛在客戶。 透過 HTTPS 端點，就能以電子郵件通知的形式送出商業市集潛在客戶，或將其寫入至 Power Automate 所支援的 CRM 系統。

本文會說明如何在 Power Automate 中建立新的流程，以產生可供用來在合作夥伴中心設定潛在客戶的 HTTP POST URL。 文章中也包含可使用 [Postman](https://www.getpostman.com/downloads/) 來測試流程的步驟。

>[!NOTE]
>若要使用在這些指示中所用 Power Automate 連接器，需具備 Power Automate 的付費訂用帳戶。 在設定此流程之前，請確定您已考量到這一點。

## <a name="create-a-flow-by-using-power-automate"></a>使用 Power Automate 建立流程

1. 開啟 [Power Automate](https://flow.microsoft.com/) 網頁。 選取 [登入]。 如果您還沒有帳戶，請選取 [免費註冊] 以建立免費的 Power Automate 帳戶。

1. 登入並選取功能表上的 [我的流程]。

    ![登入 [我的流程]](./media/commercial-marketplace-lead-management-instructions-https/my-flows-automated.png)

1. 在 [+ 新增] 底下，選取 [+ 立即 - 從頭開始]。

    ![我的流程 + 自動化--從頭開始](./media/commercial-marketplace-lead-management-instructions-https/https-myflows-create-fromblank.png)

1. 為您的流程命名，然後在 [選擇觸發此流程的方式] 底下選取 [收到 HTTP 要求時]。

    ![建置自動化流程視窗的 [略過] 按鈕](./media/commercial-marketplace-lead-management-instructions-https/https-myflows-pick-request-trigger.png)

1. 按一下流程步驟來加以展開。

    ![展開流程步驟](./media/commercial-marketplace-lead-management-instructions-https/expand-flow-step.png)

1. 使用下列其中一個方法來設定 **要求本文 JSON 結構描述**：

    - 將 JSON 結構描述複製到 [要求本文 JSON 結構描述] 文字方塊中。
    - 選取 [使用範例承載來產生結構描述]。 在 [輸入或貼上範例 JSON 承載] 文字方塊中，貼上 JSON 範例。 選取 [完成] 以建立結構描述。

    **JSON 結構描述**

    ```JSON
    {
      "$schema": "https://json-schema.org/draft-04/schema#",
      "definitions": {},
      "id": "http://example.com/example.json",
      "properties": {
        "ActionCode": {
          "id": "/properties/ActionCode",
          "type": "string"
        },
        "OfferTitle": {
          "id": "/properties/OfferTitle",
          "type": "string"
        },
        "LeadSource": {
          "id": "/properties/LeadSource",
          "type": "string"
        },
        "Description": {
          "id": "/properties/Description",
          "type": "string"
        },
        "UserDetails": {
          "id": "/properties/UserDetails",
          "properties": {
            "Company": {
              "id": "/properties/UserDetails/properties/Company",
              "type": "string"
            },
            "Country": {
              "id": "/properties/UserDetails/properties/Country",
              "type": "string"
            },
            "Email": {
              "id": "/properties/UserDetails/properties/Email",
              "type": "string"
            },
            "FirstName": {
              "id": "/properties/UserDetails/properties/FirstName",
              "type": "string"
            },
            "LastName": {
              "id": "/properties/UserDetails/properties/LastName",
              "type": "string"
            },
            "Phone": {
              "id": "/properties/UserDetails/properties/Phone",
              "type": "string"
            },
            "Title": {
              "id": "/properties/UserDetails/properties/Title",
              "type": "string"
            }
          },
          "type": "object"
        }
      },
      "type": "object"
    }
    ```

    **JSON 範例**
    
    ```json
    {
      "UserDetails": {
        "FirstName": "Some",
        "LastName": "One",
        "Email": "someone@contoso.com",
        "Phone": "16175555555",
        "Country": "USA",
        "Company": "Contoso",
        "Title": "Esquire"
     },
      "LeadSource": "AzureMarketplace",
      "ActionCode": "INS",
      "OfferTitle": "Test Microsoft",
      "Description": "Test run through Power Automate"
    }
    ```

>[!NOTE]
>此時在設定中，您可以選取要連線到 CRM 系統還是設定電子郵件通知。 根據您的選擇來遵循其餘的指示。

### <a name="connect-to-a-crm-system"></a>連線到 CRM 系統

1. 選取 [+ 新步驟] 。
1. 藉由在顯示 [搜尋連接器和動作] 的地方進行搜尋，來選擇您要的 CRM 系統。 在有動作可供建立新記錄的 [動作] 索引標籤上選取系統。 下列畫面以 **建立新記錄 (Dynamics 365)** 來舉例。

    ![建立新的記錄](./media/commercial-marketplace-lead-management-instructions-https/create-new-record.png)

1. 提供與 CRM 系統相關聯的 [組織名稱]。 從 [實體名稱] 下拉式清單選取 [潛在客戶]。

    ![選取潛在客戶](./media/commercial-marketplace-lead-management-instructions-https/select-leads.png)

1. Power Automate 會顯示一個表單來提供潛在客戶資訊。 您可以選擇新增動態內容來對應輸入要求中的項目。 下列畫面會顯示 **OfferTitle** 來作為範例。

    ![新增動態內容](./media/commercial-marketplace-lead-management-instructions-https/add-dynamic-content.png)

1. 對應您想要的欄位，然後選取 [儲存]，儲存您的流程。 HTTP POST URL 隨即建立，並且可在 [收到 HTTP 要求時] 視窗中存取。 使用複製控制項 (位於 HTTP POST URL 右邊) 來複製此 URL。 請務必使用複製控制項，如此一來，您就不會遺漏整個 URL 的任何部分。 儲存此 URL，因為當您在發佈入口網站中設定潛在客戶管理時會用到。

    ![收到 HTTP 要求時](./media/commercial-marketplace-lead-management-instructions-https/when-http-request-received.png)

### <a name="set-up-email-notification"></a>設定電子郵件通知

1. 您已完成 JSON 結構描述，接下來請選取 [+ 新增步驟]。
1. 在 [選擇動作] 底下，選取 [動作]。
1. 在 [動作] 索引標籤上，選取 [傳送電子郵件 (Office 365 Outlook)]。

    >[!NOTE]
    >如果您想要使用不同的電子郵件提供者，請改為搜尋並選取 [傳送電子郵件通知 (郵件)] 作為動作。

    ![新增電子郵件動作](./media/commercial-marketplace-lead-management-instructions-https/https-request-received-send-email.png)

1. 在 [傳送電子郵件] 視窗中，設定下列必要欄位：

   - **收件者**：請輸入至少一個有效的電子郵件地址，以供作為潛在客戶的傳送目的地。
   - **主體**：Power Automate 可讓您選擇新增動態內容，像是下列畫面所顯示的 **LeadSource**。 輸入欄位名稱來開始。 然後從快顯視窗中選取動態內容挑選清單。 

        >[!NOTE] 
        > 當您新增欄位名稱時，可以在每個名稱後面加上冒號 (:)，然後選取 **Enter** 來建立新的資料列。 欄位名稱新增好之後，您便可以從動態挑選清單新增每個相關聯的參數。

        ![使用動態內容新增電子郵件動作](./media/commercial-marketplace-lead-management-instructions-https/add-email-using-dynamic-content.png)

   - **主體**：從動態內容挑選清單中，新增您在電子郵件本文中想要的資訊。 例如，使用 LastName、FirstName、Email 和 Company。 當您完成電子郵件通知的設定時，其看起來會類似下列畫面中的範例。


       ![電子郵件通知範例](./media/commercial-marketplace-lead-management-instructions-https/send-an-email.png)

1. 選取 [儲存] 完成流程。 HTTP POST URL 隨即建立，並且可在 [收到 HTTP 要求時] 視窗中存取。 使用複製控制項 (位於 HTTP POST URL 右邊) 來複製此 URL。 請務必使用此控制項，如此一來，您就不會遺漏整個 URL 的任何部分。 儲存此 URL，因為當您在發佈入口網站中設定潛在客戶管理時會用到。

   ![HTTP POST URL](./media/commercial-marketplace-lead-management-instructions-https/http-post-url.png)

### <a name="testing"></a>測試

您可以使用 [Postman](https://app.getpostman.com/app/download/win64) 來測試您的設定。 您可以從線上下載適用於 Windows 的 Postman。 

1. 啟動 Postman，然後選取 [新增] > [要求] 來設定您的測試工具。 

   ![要求設定測試工具](./media/commercial-marketplace-lead-management-instructions-https/postman-request.png)

1. 填寫 [儲存要求] 表單，然後儲存至您所建立的資料夾。

   ![儲存要求表單](./media/commercial-marketplace-lead-management-instructions-https/postman-save-to-test.png)

1. 從下拉式清單中選取 [POST]。 

   ![測試我的流程](./media/commercial-marketplace-lead-management-instructions-https/test-my-flow.png)

1. 在顯示 [輸入要求 URL] 的地方貼上來自您在 Power Automate 中所建立流程的 HTTP POST URL。

   ![貼上 HTTP POST URL](./media/commercial-marketplace-lead-management-instructions-https/paste-http-post-url.png)

1. 返回 [Power Automate](https://flow.microsoft.com/)。 移至 Power Automate 功能表列中的 [我的流程]，尋找您為了傳送潛在客戶所建立的流程。 選取流程名稱旁的省略號以查看更多選項，然後選取 [編輯]。


1. 選取右上角的 [測試]、選取 [我會執行觸發動作]，然後選取 [測試]。 您會在畫面頂端看到測試已開始的指示。

   ![我會執行觸發動作選項](./media/commercial-marketplace-lead-management-instructions-https/test-flow-trigger-action.png)

1. 返回 Postman 應用程式，然後選取 [傳送]。

   ![[傳送] 按鈕](./media/commercial-marketplace-lead-management-instructions-https/postman-send.png)

1. 返回您的流程並檢查結果。 如果一切順利，您會看到訊息指出流程已成功。

   ![檢查結果](./media/commercial-marketplace-lead-management-instructions-https/my-flow-check-results.png)

1. 您也應該會收到一封電子郵件。 檢查您的電子郵件收件匣。 

    >[!NOTE] 
    >如果您沒有看到來自測試的電子郵件，請檢查您的垃圾郵件資料夾。 在下列畫面中，您只會看到設定電子郵件通知時所新增的欄位標籤。 如果這是從您的供應項目產生的實際潛在客戶，則您還會在本文與主旨行中看到潛在客戶連絡人的實際資訊。

   ![收到的電子郵件](./media/commercial-marketplace-lead-management-instructions-https/email-received.png)

## <a name="configure-your-offer-to-send-leads-to-the-https-endpoint"></a>設定您的供應項目，以便將潛在客戶傳送至 HTTPS 端點

當您準備好在發佈入口網站中設定供應項目的潛在客戶管理資訊時，請遵循下列步驟。

1. 登入 [合作夥伴中心](https://partner.microsoft.com/dashboard/home)。

1. 選取您的供應項目，然後移至 [供應項目設定] 索引標籤。

1. 在 [潛在客戶] 區段底下，選取 [連線]。

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-https/customer-leads.png" alt-text="潛在客戶":::

1. 在 [連線詳細資料] 快顯視窗上，選取 [HTTPS 端點] 作為 [潛在客戶目的地]。 將您遵循先前步驟所建立的流程所提供的 HTTP POST URL 貼到 [HTTPS 端點 URL] 欄位中。
    ![連線詳細資料連絡人電子郵件](./media/commercial-marketplace-lead-management-instructions-https/https-connection-details.png)

1. 在 [連絡人電子郵件] 下輸入公司收到新的潛在客戶時，會收到電子郵件通知之員工的電子郵件地址。 您可以提供多個電子郵件，方法是以分號分隔。

1. 選取 [確定]。

為確定您已成功連線到潛在客戶目的地，請選取 [驗證] 按鈕。 如果成功，即會在潛在客戶目的地中看到測試潛在客戶。

>[!NOTE] 
>您必須完成供應項目的其餘設定並加以發佈，才能接收到該供應項目的潛在客戶。

產生潛在客戶時，Microsoft 便會將該潛在客戶傳送到流程中。 潛在客戶會路由傳送至您所設定的 CRM 系統或電子郵件地址。
