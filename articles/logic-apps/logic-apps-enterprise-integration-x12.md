---
title: 適用於 B2B 企業整合的 X12 訊息 - Azure Logic Apps | Microsoft Docs
description: 在採用 Enterprise Integration Pack 的 Azure Logic Apps 中交換適用於 B2B 企業整合的 X12 訊息 (採用 EDI 格式)
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 7422d2d5-b1c7-4a11-8c9b-0d8cfa463164
ms.date: 01/31/2017
ms.openlocfilehash: c4ee56f4ddcccb1fc4ddd84aa1c1b16dea9754d9
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123952"
---
# <a name="exchange-x12-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>在採用 Enterprise Integration Pack 的 Azure Logic Apps 中交換適用於 B2B 企業整合的 X12 訊息

您必須先建立 X12 合約並將該合約儲存在您的整合帳戶中，才可以交換 X12 訊息。 以下是如何建立 X12 合約的步驟。

> [!NOTE]
> 本頁涵蓋 Azure Logic Apps 的 X12 功能。 如需詳細資訊，請參閱 [EDIFACT](logic-apps-enterprise-integration-edifact.md)。

## <a name="before-you-start"></a>開始之前

以下是您所需的項目︰

* 已經定義並與 Azure 訂用帳戶相關聯的[整合帳戶](logic-apps-enterprise-integration-create-integration-account.md)
* 至少已在整合帳戶中定義兩個[夥伴](../logic-apps/logic-apps-enterprise-integration-partners.md)，以及在 [企業身分識別] 之下設定 X12 識別碼    
* 您可以上傳到整合帳戶的必要[結構描述](../logic-apps/logic-apps-enterprise-integration-schemas.md)

在您[建立整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)、[新增夥伴](logic-apps-enterprise-integration-partners.md)，以及具有您有使用的[結構描述](../logic-apps/logic-apps-enterprise-integration-schemas.md)之後，您可以依照下列步驟建立 X12 合約。

## <a name="create-an-x12-agreement"></a>建立 X12 合約

1. 登入 [Azure 入口網站](http://portal.azure.com "Azure 入口網站")。 

2. 從主要 Azure 功能表中選取 [所有服務]。 在搜尋方塊中輸入「整合」，然後選取 [整合帳戶]。  

   ![尋找您的整合帳戶](./media/logic-apps-enterprise-integration-x12/account-1.png)

   > [!TIP]
   > 如果 [所有服務] 未出現，您可能必須先展開功能表。 在摺疊功能表的頂端，選取 [顯示功能表]。

3. 在 [整合帳戶] 底下，選取要在其中新增合約的整合帳戶。

   ![選取您要在其中建立合約的整合帳戶](./media/logic-apps-enterprise-integration-x12/account-3.png)

4. 選取 [概觀]，然後選取 [合約] 圖格。 如果您沒有 [合約] 圖格，請先新增圖格。 

   ![選擇 [合約] 圖格](./media/logic-apps-enterprise-integration-as2/agreement-1.png)

5. 在 [合約] 之下，選擇 [新增]。

   ![選擇 [新增]](./media/logic-apps-enterprise-integration-as2/agreement-2.png)     

6. 在 [新增] 之下，輸入合約的 [名稱]。 針對 [合約類型]，選取 **X12**。 選取合約的 [主機夥伴]、[主機身分識別]、[來賓夥伴] 和 [來賓身分識別]。 如需更多屬性詳細資訊，請參閱此步驟中的資料表。

    ![提供合約詳細資料](./media/logic-apps-enterprise-integration-x12/x12-1.png)  

    | 屬性 | 說明 |
    | --- | --- |
    | 名稱 |合約的名稱 |
    | 合約類型 | 應該是 X12 |
    | 主控夥伴 |合約需要主控夥伴和來賓夥伴。 主機夥伴代表設定合約的組織。 |
    | 主控身分識別 |主控夥伴的識別碼 |
    | 來賓夥伴 |合約需要主控夥伴和來賓夥伴。 來賓夥伴代表與主控夥伴有生意往來的組織。 |
    | 來賓身分識別 |來賓合作夥伴的識別碼 |
    | 接收設定 |這些屬性會套用到合約所接收的所有訊息。 |
    | 傳送設定 |這些屬性會套用到合約所傳送的所有訊息。 |  

  > [!NOTE]
  > X12 合約的解析取決於比對合作夥伴與內送郵件中定義的傳送者辨識符號與識別碼，以及接收者辨識符號與識別碼。 如果您夥伴的這些值變更，請同時更新合約。

## <a name="configure-how-your-agreement-handles-received-messages"></a>設定合約處理所收到訊息的方式

您現在已經設定合約屬性，您可以設定此合約如何識別及處理您透過此合約從夥伴接收的內送訊息。

1.  在 [新增] 之下，選取 [接收設定]。
根據您與其交換訊息之夥伴所簽署的合約，設定這些屬性。 如需屬性說明，請參閱本節中的資料表。

    [接收設定] 分成下列各區段：識別碼、通知、結構描述、信封、控制編號、驗證和內部設定。

2. 完成後，請務必選擇 [確定] 以儲存設定。

您的合約現在已準備好處理符合您所選設定的內送訊息。

### <a name="identifiers"></a>識別項

![設定識別碼屬性](./media/logic-apps-enterprise-integration-x12/x12-2.png)  

| 屬性 | 說明 |
| --- | --- |
| ISA1 (授權辨識符號) |從下拉式清單中選取授權辨識符號值。 |
| ISA2 |選用。 輸入授權資訊值。 如果您為 ISA1 輸入的值是 00 以外的值，請最少輸入 1 個，最多 10 個英數字元。 |
| ISA3 (安全性辨識符號) |從下拉式清單中選取安全性辨識符號值。 |
| ISA4 |選用。 輸入安全性資訊值。 如果您為 ISA3 輸入的值是 00 以外的值，請最少輸入 1 個，最多 10 個英數字元。 |

### <a name="acknowledgment"></a>通知

![設定通知屬性](./media/logic-apps-enterprise-integration-x12/x12-3.png) 

| 屬性 | 說明 |
| --- | --- |
| 預期 TA1 |將技術通知傳回給交換傳送者 |
| 預期 FA |將功能通知傳回給交換傳送者。 接著，根據結構描述版本，選取您想要 997 或 999 通知 |
| 包含 AK2/IK2 迴圈 |在功能通知中針對已接受的交易集啟用 AK2 迴圈的產生 |

### <a name="schemas"></a>結構描述

選取每個交易類型 (ST1) 和傳送者應用程式 (GS2) 的結構描述。 接收管線會比對內送訊息中 ST1 和 GS2 的值與您在此處設定的值，以及比對內送訊息的結構描述與您在此處設定的結構描述，來解譯內送訊息。

![選取結構描述](./media/logic-apps-enterprise-integration-x12/x12-33.png) 

| 屬性 | 說明 |
| --- | --- |
| 版本 |選取 X12 版本 |
| 交易類型 (ST01) |選取交易類型 |
| 傳送者應用程式 (GS02) |選取傳送者應用程式 |
| 結構描述 |選取您要使用的結構描述檔案。 結構描述已新增到您的整合帳戶。 |

> [!NOTE]
> 設定要上傳到[整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md)的必要[結構描述](../logic-apps/logic-apps-enterprise-integration-schemas.md)。

### <a name="envelopes"></a>信封

![指定交易集中的分隔符號︰選擇 [標準識別碼] 或 [重複分隔符號]](./media/logic-apps-enterprise-integration-x12/x12-34.png)

| 屬性 | 說明 |
| --- | --- |
| ISA11 使用方法 |指定要用於交易集中的分隔符號︰ <p>選取 [標準識別碼] 以使用句號 (.) 做為小數點標記，而不是 EDI 接收管線中內送文件的小數點標記。 <p>選取 [重複分隔符號]，以指定重複出現簡單資料元素或重複資料結構的分隔符號。 例如，插入號 (^) 通常會做為重複分隔符號。 針對 HIPAA 結構描述，您只能使用插入號。 |

### <a name="control-numbers"></a>控制編號

![選取如何處理控制編號重複項目](./media/logic-apps-enterprise-integration-x12/x12-35.png) 

| 屬性 | 說明 |
| --- | --- |
| 不允許交換控制編號重複 |封鎖重複的交換。 檢查已接收之交換控制編號的交換控制編號 (ISA13)。 如果偵測到相符項目，接收管線就不會處理交換。 您可以提供 [檢查 ISA13 是否重複的天數間隔] 的值，以指定執行檢查的天數。 |
| 不允許群組控制編號重複 |封鎖有重複群組控制編號的交換。 |
| 不允許交易集控制編號重複 |封鎖有重複交易集控制編號的交換。 |

### <a name="validations"></a>驗證

![設定已接收訊息的驗證屬性](./media/logic-apps-enterprise-integration-x12/x12-36.png) 

當您完成每個驗證資料列時，將會自動新增另一個驗證資料列。 如果您未指定任何規則，則驗證會使用「預設」資料列。

| 屬性 | 說明 |
| --- | --- |
| 訊息類型 |選取 EDI 訊息類型。 |
| EDI 驗證 |依照結構描述的 EDI 屬性、長度限制、空白資料元素和尾端分隔符號定義，在資料類型上執行 EDI 驗證。 |
| 擴充驗證 |如果資料類型不是 EDI，則在資料元素要求時才進行驗證，且允許重複、列舉和資料元素長度驗證 (最小/最大)。 |
| 允許前置/尾端零 |保留任何額外的前置或尾端零及空格字元。 請勿移除這些字元。 |
| 修剪前置/尾端零 |移除前置或尾端零及空格字元。 |
| 尾端分隔符號原則 |產生尾端分隔符號。 <p>選取 [不允許]，禁止在已接收的交換中使用尾端分隔符號。 如果交換具有尾端分隔符號，則會被宣告為無效。 <p>選取 [選擇性]  以接受包含或不含尾端分隔符號的交換。 <p>如果交換必須具備尾端分隔符號，請選取 [強制性]。 |

### <a name="internal-settings"></a>內部設定

![選取內部設定](./media/logic-apps-enterprise-integration-x12/x12-37.png) 

| 屬性 | 說明 |
| --- | --- |
| 將隱含的小數格式 "Nn" 轉換為 10 進制數值 |將使用 "Nn" 格式指定的 EDI 數字轉換為 10 進制數值 |
| 如果允許尾端分隔符號，則建立空的 XML 標籤 |選取此核取方塊，讓交換傳送者在尾端分隔符號包含空的 XML 標籤。 |
| 將交換分割為交易集 - 發生錯誤時暫停交易集|套用適當的信封至交易集，將交換中每個交易集剖析為個別的 XML 文件。 只暫停驗證失敗的交易。 |
| 將交換分割為交易集 - 發生錯誤時暫停交換|套用適當的信封，將交換中每個交易集剖析為個別的 XML 文件。 如果交換中有一或多個交易集無法通過驗證，則會暫停整個交換。 | 
| 保留交換 - 發生錯誤時暫停交易集 |讓交換維持不變，建立整個批次交換的 XML 文件。 只暫停未通過驗證的交易集，而繼續處理所有其他交易集。 |
| 保留交換 - 發生錯誤時暫停交換 |讓交換維持不變，建立整個批次交換的 XML 文件。 如果交換中有一或多個交易集無法通過驗證，則會暫停整個交換。 |

## <a name="configure-how-your-agreement-sends-messages"></a>設定合約傳送訊息的方式

您可以設定此合約如何識別及處理您透過此合約傳送給夥伴的外寄訊息。

1.  在 [新增] 之下，選取 [傳送設定]。
根據您與其交換訊息之夥伴所簽署的合約，設定這些屬性。 如需屬性說明，請參閱本節中的資料表。

    [傳送設定] 分成下列各區段：識別碼、通知、結構描述、信封、字元集和分隔符號、控制編號以及驗證。

2. 完成後，請務必選擇 [確定] 以儲存設定。

您的合約現在已準備好處理符合您所選設定的外寄訊息。

### <a name="identifiers"></a>識別項

![設定識別碼屬性](./media/logic-apps-enterprise-integration-x12/x12-4.png)  

| 屬性 | 說明 |
| --- | --- |
| 授權辨識符號 (ISA1) |從下拉式清單中選取授權辨識符號值。 |
| ISA2 |輸入授權資訊值。 如果此值不是 00，則請最少輸入 1 個，最多 10 個英數字元。 |
| 安全性辨識符號 (ISA3) |從下拉式清單中選取安全性辨識符號值。 |
| ISA4 |輸入安全性資訊值。 如果此值不是 00，則在 [值 (ISA4)] 文字方塊中，請最少輸入 1 個，最多 10 個英數字元值。 |

### <a name="acknowledgment"></a>通知

![設定通知屬性](./media/logic-apps-enterprise-integration-x12/x12-5.png)  

| 屬性 | 說明 |
| --- | --- |
| 預期 TA1 |將技術通知 (TA1) 傳回給交換傳送者。 此設定指定傳送訊息的主控夥伴向合約中的來賓合作夥伴要求通知。 主控夥伴根據合約的「接收設定」，預期有這些通知。 |
| 預期 FA |將功能通知 (FA) 傳回給交換傳送者。 根據您正在使用的結構描述版本，選取您想要 997 或 999 通知。 主控夥伴根據合約的「接收設定」，預期有這些通知。 |
| FA 版本 |選取 FA 版本 |

### <a name="schemas"></a>結構描述

![選取要使用的結構描述](./media/logic-apps-enterprise-integration-x12/x12-5.png)  

| 屬性 | 說明 |
| --- | --- |
| 版本 |選取 X12 版本 |
| 交易類型 (ST01) |選取交易類型 |
| 結構描述 |選取要使用的結構描述。 結構描述位於您的整合帳戶中。 若先選取結構描述，它會自動設定版本與交易類型  |

> [!NOTE]
> 設定要上傳到[整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md)的必要[結構描述](../logic-apps/logic-apps-enterprise-integration-schemas.md)。

### <a name="envelopes"></a>信封

![指定交易集中的分隔符號︰選擇 [標準識別碼] 或 [重複分隔符號]](./media/logic-apps-enterprise-integration-x12/x12-6.png) 

| 屬性 | 說明 |
| --- | --- |
| ISA11 使用方法 |指定要用於交易集中的分隔符號︰ <p>選取 [標準識別碼] 以使用句號 (.) 做為小數點標記，而不是 EDI 接收管線中內送文件的小數點標記。 <p>選取 [重複分隔符號]，以指定重複出現簡單資料元素或重複資料結構的分隔符號。 例如，插入號 (^) 通常會做為重複分隔符號。 針對 HIPAA 結構描述，您只能使用插入號。 |

### <a name="control-numbers"></a>控制編號

![指定控制編號屬性](./media/logic-apps-enterprise-integration-x12/x12-8.png) 

| 屬性 | 說明 |
| --- | --- |
| 控制版本編號 (ISA12) |選取 X12 標準的版本 |
| 使用狀況指示符號 (ISA15) |選取交換的內容。  值是資訊、生產資料或測試資料 |
| 結構描述 |產生 X12 編碼交換的 GS 和 ST 區段，它會將這些區段傳送至「傳送管線」 |
| GS1 |選擇性，請從下拉式清單選取功能代碼的值 |
| GS2 |選擇性，應用程式寄件者 |
| GS3 |選擇性，應用程式接收者 |
| GS4 |選擇性，請選取 CCYYMMDD 或 YYMMDD |
| GS5 |選擇性，請選取 HHMM、HHMMSS 或 HHMMSSdd |
| GS7 |選擇性，請從下拉式清單選取負責單位的值 |
| GS8 |選擇性，文件的版本 |
| 交換控制編號 (ISA13) |必要，輸入交換控制編號的值範圍。 請輸入最小為 1，最大為 999999999 的數值 |
| 群組控制編號 (GS06) |必要，請輸入群組控制編號的數字範圍。 請輸入最小為 1，最大為 999999999 的數值 |
| 交易集控制編號 (ST02) |必要，請輸入交易集控制編號的數字範圍。 請輸入最小為 1，最大為 999999999 的數值範圍 |
| 前置詞 |選擇性，針對在通知中使用的交易集控制編號範圍指定。 在中間的兩個欄位輸入數值，在前置詞和尾碼欄位輸入英數字元值 (如有需要)。 中間欄位是必要欄位，而且包含控制編號的最小值和最大值 |
| 尾碼 |選擇性，針對在通知中使用的交易集控制編號範圍指定。 在中間的兩個欄位輸入數值，在前置詞和尾碼欄位輸入英數字元值 (如有需要)。 中間欄位是必要欄位，而且包含控制編號的最小值和最大值 |

### <a name="character-sets-and-separators"></a>字元集和分隔符號

除了字元集，您還可以為每個訊息類型輸入一組不同的分隔符號。 如果指定的訊息結構描述未指定字元集，則會使用預設字元集。

![指定不同訊息類型的分隔符號](./media/logic-apps-enterprise-integration-x12/x12-9.png) 

| 屬性 | 說明 |
| --- | --- |
| 要使用的字元集 |若要驗證屬性，請選取 X12 字元集。 選項包括 [基本]、[延伸] 和 [UTF8]。 |
| 結構描述 |從下拉式清單中選取結構描述。 在您完成每個資料列時，便會自動新增一個資料列。 針對選取的結構描述，根據下面的分隔符號說明，選取您要使用的分隔符號集。 |
| 輸入類型 |從下拉式清單中選取輸入類型。 |
| 元件分隔符號 |若要分隔複合的資料元素，請輸入單一字元。 |
| 資料元素分隔符號 |若要分隔複合資料元素內的簡單資料元素，請輸入單一字元。 |
| 取代字元 |輸入在產生輸出 X12 訊息時，用於取代承載資料中所有分隔符號字元的取代字元。 |
| 區段結束字元 |若要指出 EDI 區段的結尾，請輸入單一字元。 |
| 尾碼 |選取與區段識別項一起使用的字元。 如果指定尾碼，則區段結束字元資料元素可以空白。 如果區段結束字元留空，則必須指定尾碼。 |

> [!TIP]
> 若要提供特殊字元值，請將合約編輯為 JSON，並為特殊字元提供 ASCII 值。

### <a name="validation"></a>驗證

![設定驗證屬性以便傳送訊息](./media/logic-apps-enterprise-integration-x12/x12-10.png) 

當您完成每個驗證資料列時，將會自動新增另一個驗證資料列。 如果您未指定任何規則，則驗證會使用「預設」資料列。

| 屬性 | 說明 |
| --- | --- |
| 訊息類型 |選取 EDI 訊息類型。 |
| EDI 驗證 |依照結構描述的 EDI 屬性、長度限制、空白資料元素和尾端分隔符號定義，在資料類型上執行 EDI 驗證。 |
| 擴充驗證 |如果資料類型不是 EDI，則在資料元素要求時才進行驗證，且允許重複、列舉和資料元素長度驗證 (最小/最大)。 |
| 允許前置/尾端零 |保留任何額外的前置或尾端零及空格字元。 請勿移除這些字元。 |
| 修剪前置/尾端零 |移除前置或尾端零字元。 |
| 尾端分隔符號原則 |產生尾端分隔符號。 <p>選取 [不允許]，禁止在已傳送的交換中使用尾端分隔符號。 如果交換具有尾端分隔符號，則會被宣告為無效。 <p>選取 [選擇性] 以傳送包含或不含尾端分隔符號的交換。 <p>如果傳送的交換必須具備尾端分隔符號，請選取 [強制性]。 |

## <a name="find-your-created-agreement"></a>尋找您建立的合約

1.  完成所有合約屬性的設定之後，請在 [新增] 頁面中選擇 [確定]，以完成合約建立並回到整合帳戶。

    您新增的合約現在顯示於您的 [合約] 清單中。

2.  您也可以在整合帳戶概觀中檢視您的合約。 在整合帳戶功能表上，選擇 [概觀]，然後選取 [合約] 圖格。

    ![選擇 [合約] 圖格](./media/logic-apps-enterprise-integration-x12/x12-1-5.png)   

## <a name="view-the-swagger"></a>檢視 Swagger
請參閱 [Swagger 詳細資料](/connectors/x12/)。 

## <a name="learn-more"></a>深入了解
* [深入了解企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md "了解企業整合套件")  

