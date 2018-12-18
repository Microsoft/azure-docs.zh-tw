---
title: Azure 資料目錄開發人員概念
description: Azure 資料目錄概念模型的重要概念簡介，以透過目錄 REST API 的形式公開。
services: data-catalog
author: markingmyname
ms.author: maghan
ms.assetid: 89de9137-a0a4-40d1-9f8d-625acad31619
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 97dafe01db4779332a530dc8981b3d6ada8c9df9
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406510"
---
# <a name="azure-data-catalog-developer-concepts"></a>Azure 資料目錄開發人員概念
Microsoft **Azure 資料目錄** 是完全受控的雲端服務，能夠進行資料來源探索，以及讓群眾外包資料來源中繼資料。 開發人員可以透過 REST API 來使用此服務。 開發人員必須了解服務中所實作的概念，才能成功地與 **Azure 資料目錄**整合。

## <a name="key-concepts"></a>重要概念
**Azure 資料目錄**概念模型根據四個主要概念：**目錄**、**使用者**、**資產**和**註解**。

![概念][1]

*圖 1 - Azure 資料目錄簡易概念模型*

### <a name="catalog"></a>目錄
**目錄** 是組織所儲存之所有中繼資料的最上層容器。 每個 Azure 帳戶只能一個 **目錄** 。 目錄與 Azure 訂用帳戶密切相關，即使一個帳戶可以有多個訂用帳戶，但任一特定 Azure 帳戶只能建立一個 **目錄** 。

目錄包含**使用者**和**資產**。

### <a name="users"></a>使用者
使用者是有權在目錄中執行動作的安全性主體 (搜尋目錄、新增、編輯或移除項目等等)。

一個使用者可能扮演多個不同的角色。 如需角色的相關資訊，請參閱＜角色和授權＞一節。

可加入個別的使用者和安全性群組。

Azure 資料目錄使用 Azure Active Directory 來管理身分識別和存取權。 每個目錄使用者必須是帳戶 Active Directory 的成員。

### <a name="assets"></a>Assets
**目錄** 包含資料資產。 **資產** 是由目錄管理的細微度單位。

資產的細微度隨資料來源而異。 對於 SQL Server 或 Oracle 資料庫，資產可以是資料表或檢視。 對於 SQL Server Analysis Services，資產可以是量值、維度或關鍵效能指標 (KPI)。 對於 SQL Server Reporting Services，資產是報表。

**資產** 是您在目錄中加入或移除的項目。 它是您從 **搜尋**取回的結果單位。

**資產** 由其名稱、位置和類型，以及進一步描述它的註解所組成。

### <a name="annotations"></a>註解
註解是代表資產中繼資料的項目。

註解的範例為描述、標籤、結構描述、文件等等。＜資產物件模型＞一節中有資產類型和註解類型的完整清單。

## <a name="crowdsourcing-annotations-and-user-perspective-multiplicity-of-opinion"></a>群眾外包註解和使用者觀點 (多樣性意見)
Azure 資料目錄的重點在於如何支援由群眾外包系統中的中繼資料。 與 Wiki 的作法不同 – 其中只有一個意見，以最後一個寫入者為準 – Azure 資料目錄模型允許多種意見並存於系統中。

這種作法反映出企業資料的真實情況，不同的使用者對特定的資產可以有不同的觀點：

* 資料庫管理員可以提供服務等級協定或可用於處理大量 ETL 作業的時段的相關資訊
* 資料負責人可以提供資產適用的商務程序或企業採用的分類辦法的相關資訊
* 財務分析師可以提供期末報告工作期間如何使用資料的相關資訊

為了支援這個例子，每位使用者 (DBA、資料負責人和分析師) 可以將描述加入至目錄中已註冊的資料表。 所有描述會都保留在系統中，Azure 資料目錄入口網站會顯示所有的描述。

此模式適用於物件模型中的大部分項目，因此您可能以為 JSON 承載中的物件類型只有一個屬性，但實際上經常有大量屬性。

例如，資產根目錄底下有大量的描述物件。 此陣列屬性稱為 “descriptions”。 描述物件有一個屬性，也就是描述。 模式是輸入描述的每個使用者都會取得針對使用者提供的值所建立的描述物件。

然後 UX 可以選擇如何顯示組合。 有三種不同的顯示模式。

* 最簡單的模式為 [全部顯示]。 在此模式下，所有物件會以清單檢視來顯示。 Azure 資料目錄入口網站 UX 會對描述使用此模式。
* 另一種模式是「合併」。 在此模式下，不同使用者提供的所有值會合併在一起，並移除重複項目。 Azure 資料目錄入口網站 UX 中的 tags 和 experts 屬性，即為此模式的例子。
* 第三種模式為「最後寫入者為準」。 在此模式下，只會顯示最後輸入的值。 friendlyName 是這種模式的一個例子。

## <a name="asset-object-model"></a>資產物件模型
＜重要概念＞一節介紹的 **Azure 資料目錄** 物件模型所包含的項目，可以是資產或註解。 項目具有選用或必要的屬性。 某些屬性會套用至所有項目。 某些屬性會套用至所有資產。 某些屬性只套用至特定的資產類型。

### <a name="system-properties"></a>系統屬性
<table><tr><td><b>屬性名稱</b></td><td><b>資料類型</b></td><td><b>註解</b></td></tr><tr><td>timestamp</td><td>Datetime</td><td>上次修改項目的時間。 伺服器會在插入項目時以及每次更新項目時產生此欄位。 此屬性值會在輸入發佈作業時遭到忽略。</td></tr><tr><td>id</td><td>Uri</td><td>項目的絕對 URL (唯讀)。 它是項目的唯一可定址 URI。  此屬性值會在輸入發佈作業時遭到忽略。</td></tr><tr><td>type</td><td>字串</td><td>資產的類型 (唯讀)。</td></tr><tr><td>etag</td><td>字串</td><td>對應到項目版本的字串，在執行會更新目錄中項目的作業時可用於開放式並行存取控制。 "*" 可用來比對任何值。</td></tr></table>

### <a name="common-properties"></a>通用屬性
這些屬性套用至所有根資產類型和所有註解類型。

<table>
<tr><td><b>屬性名稱</b></td><td><b>資料類型</b></td><td><b>註解</b></td></tr>
<tr><td>fromSourceSystem</td><td>BOOLEAN</td><td>表示項目的資料是衍生自來源系統 (例如 SQL Server Database、Oracle Database) 或由使用者所撰寫。</td></tr>
</table>

### <a name="common-root-properties"></a>通用根屬性
<p>
這些屬性套用至所有根資產類型。

<table><tr><td><b>屬性名稱</b></td><td><b>資料類型</b></td><td><b>註解</b></td></tr><tr><td>name</td><td>字串</td><td>衍生自資料來源位置資訊的名稱</td></tr><tr><td>dsl</td><td>DataSourceLocation</td><td>可唯一描述資料來源，為資產的其中一個識別碼 (請參閱＜雙重識別＞一節)。  dsl 的結構隨通訊協定和來源類型而異。</td></tr><tr><td>dataSource</td><td>DataSourceInfo</td><td>資產類型的詳細資料。</td></tr><tr><td>lastRegisteredBy</td><td>SecurityPrincipal</td><td>描述最近註冊此資產的使用者。  包含使用者的唯一識別碼 (upn) 和顯示名稱 (lastName 和 firstName)。</td></tr><tr><td>containerId</td><td>字串</td><td>資料來源的容器資產識別碼。 容器類型不支援這個屬性。</td></tr></table>

### <a name="common-non-singleton-annotation-properties"></a>一般的非單一註解屬性
這些屬性套用至所有非單一註解類型 (每個資產允許有多個的註解)。

<table>
<tr><td><b>屬性名稱</b></td><td><b>資料類型</b></td><td><b>註解</b></td></tr>
<tr><td>索引鍵</td><td>字串</td><td>可唯一識別目前集合中之註解的使用者指定索引鍵。 索引鍵長度不能超過 256 個字元。</td></tr>
</table>

### <a name="root-asset-types"></a>根資產類型
根資產類型所指的類型代表可以註冊在目錄中的各種資料資產。 每個根類型都有一個檢視，可描述檢視中包含的資產和註解。 使用 REST API 發佈資產時，檢視名稱應用於對應 {view_name} url 區段。

<table><tr><td><b>資產類型 (檢視名稱)</b></td><td><b>其他屬性</b></td><td><b>資料類型</b></td><td><b>允許的註解</b></td><td><b>註解</b></td></tr><tr><td>資料表 ("tables")</td><td></td><td></td><td>說明<p>FriendlyName<p>Tag<p>結構描述<p>ColumnDescription<p>ColumnTag<p> 專家<p>預覽<p>AccessInstruction<p>TableDataProfile<p>ColumnDataProfile<p>ColumnDataClassification<p>文件<p></td><td>資料表代表任何表格式資料。  例如：SQL 資料表、SQL 檢視、Analysis Services 表格式資料表、Analysis Services 多維度的維度、Oracle 資料表等等。   </td></tr><tr><td>量值 ("measures")</td><td></td><td></td><td>說明<p>FriendlyName<p>Tag<p>專家<p>AccessInstruction<p>文件<p></td><td>此類型代表 Analysis Services 量值。</td></tr><tr><td></td><td>measure</td><td>欄</td><td></td><td>描述量值的中繼資料</td></tr><tr><td></td><td>isCalculated </td><td>BOOLEAN</td><td></td><td>指定是否計算量值。</td></tr><tr><td></td><td>measureGroup</td><td>字串</td><td></td><td>量值的實體容器</td></tr><td>KPI ("kpis")</td><td></td><td></td><td>說明<p>FriendlyName<p>Tag<p>專家<p>AccessInstruction<p>文件</td><td></td></tr><tr><td></td><td>measureGroup</td><td>字串</td><td></td><td>量值的實體容器</td></tr><tr><td></td><td>goalExpression</td><td>字串</td><td></td><td>會傳回 KPI 目標值的 MDX 數值運算式或計算。</td></tr><tr><td></td><td>valueExpression</td><td>字串</td><td></td><td>會傳回 KPI 實際值的 MDX 數值運算式。</td></tr><tr><td></td><td>statusExpression</td><td>字串</td><td></td><td>代表指定時間點之 KPI 狀態的 MDX 運算式。</td></tr><tr><td></td><td>trendExpression</td><td>字串</td><td></td><td>評估一段時間的 KPI 值的 MDX 運算式。 趨勢可以是特定商務情況下適用的任何時間性準則。</td>
<tr><td>報表 ("reports")</td><td></td><td></td><td>說明<p>FriendlyName<p>Tag<p>專家<p>AccessInstruction<p>文件<p></td><td>此類型代表 SQL Server Reporting Services 報表 </td></tr><tr><td></td><td>assetCreatedDate</td><td>字串</td><td></td><td></td></tr><tr><td></td><td>assetCreatedBy</td><td>字串</td><td></td><td></td></tr><tr><td></td><td>assetModifiedDate</td><td>字串</td><td></td><td></td></tr><tr><td></td><td>assetModifiedBy</td><td>字串</td><td></td><td></td></tr><tr><td>容器 ("containers")</td><td></td><td></td><td>說明<p>FriendlyName<p>Tag<p>專家<p>AccessInstruction<p>文件<p></td><td>此類型代表其他資產 (例如 SQL database、Azure Blob 容器或 Analysis Services 模型) 的容器 。</td></tr></table>

### <a name="annotation-types"></a>註解類型
註解類型代表可以指派給目錄內其他類型的中繼資料類型。

<table>
<tr><td><b>註解類型 (巢狀檢視名稱)</b></td><td><b>其他屬性</b></td><td><b>資料類型</b></td><td><b>註解</b></td></tr>

<tr><td>描述 ("descriptions")</td><td></td><td></td><td>此屬性包含資產的描述。 系統的每個使用者可以加入自己的描述。  只有該使用者可以編輯 Description 物件  (系統管理員和資產擁有者可以刪除 Description 物件，但無法編輯它)。 系統會個別維護使用者的描述。  因此，每個資產上有一個描述陣列 (除了可能有一個描述包含衍生自資料來源的資訊，每一個已對資產貢獻知識的使用者都有一個描述)。</td></tr>
<tr><td></td><td>說明</td><td>字串</td><td>資產的簡短描述 (2-3 行)</td></tr>

<tr><td>標籤 ("tags")</td><td></td><td></td><td>這個屬性會定義資產的標籤。 系統的每個使用者可以為資產新增多個標記。  只有建立 Tag 物件的使用者可以編輯它們。  (系統管理員和資產擁有者可以刪除 Tag 物件，但無法編輯它)。 系統會個別維護使用者的標籤。  因此每個資產上都會有 Tag 物件的陣列。</td></tr>
<tr><td></td><td>tag</td><td>字串</td><td>描述資產的標記。</td></tr>

<tr><td>FriendlyName ("friendlyName")</td><td></td><td></td><td>此屬性包含資產的易記名稱。 FriendlyName 是單一註解 - 資產只能新增一個 FriendlyName。  只有建立 FriendlyName 物件的使用者可以編輯它。 (系統管理員和資產擁有者可以刪除 FriendlyName 物件，但無法編輯它)。 系統會個別維護使用者的易記名稱。</td></tr>
<tr><td></td><td>friendlyName</td><td>字串</td><td>資產的易記名稱。</td></tr>

<tr><td>結構描述 ("schema")</td><td></td><td></td><td>Schema 描述資料的結構。  它會列出屬性 (資料行、屬性、欄位等等) 名稱、類型及其他中繼資料。  此資訊完全衍生自資料來源。  Schema 是單一註解 - 資產只能新增一個 Schema。</td></tr>
<tr><td></td><td>columns</td><td>Column[]</td><td>資料行物件的陣列。 它們以衍生自資料來源的資訊來描述資料行。</td></tr>

<tr><td>ColumnDescription ("columnDescriptions")</td><td></td><td></td><td>此屬性包含資料行的描述。  系統的每個使用者可以為多個資料行加入自己的描述 (每個資料行最多一個)。 只有建立 ColumnDescription 物件的使用者可以編輯它們。  (系統管理員和資產擁有者可以刪除 ColumnDescription 物件，但無法編輯它)。 系統會個別維護這些使用者的資料行描述。  因此，每個資產上有一個 ColumnDescription 物件陣列 (除了可能有一個描述包含衍生自資料來源的資訊，每一個已對資料行貢獻知識的使用者都會有一個資料行描述)。  ColumnDescription 與結構描述不緊密繫結，所以可能不會同步。ColumnDescription 描述的資料行可能已不存在於結構描述中。  由寫入者決定是否要讓描述和結構描述保持同步。資料來源也可能有資料行描述資訊，這些資料行是執行工具時建立的其他 ColumnDescription 物件。</td></tr>
<tr><td></td><td>columnName</td><td>字串</td><td>此描述所參考的資料行名稱。</td></tr>
<tr><td></td><td>說明</td><td>字串</td><td>資料行的簡短描述 (2-3 行)。</td></tr>

<tr><td>ColumnTag ("columnTags")</td><td></td><td></td><td>此屬性包含資料行的標籤。 系統的每個使用者可以為特定資料行新增多個標記，也可以為多個資料行新增標記。 只有建立 ColumnTag 物件的使用者可以編輯它們。 (系統管理員和資產擁有者可以刪除 ColumnTag 物件，但無法編輯它)。 系統會個別維護這些使用者的資料行標籤。  因此每個資產上都會有 ColumnTag 物件的陣列。  ColumnTag 與結構描述不緊密繫結，所以可能不會同步。ColumnTag 描述的資料行可能已不存在於結構描述中。  由寫入者決定是否要讓資料行標籤和結構描述保持同步。</td></tr>
<tr><td></td><td>columnName</td><td>字串</td><td>此標記所參考的資料行名稱。</td></tr>
<tr><td></td><td>tag</td><td>字串</td><td>描述資料行的標記。</td></tr>

<tr><td>專家 ("experts")</td><td></td><td></td><td>此屬性包含資料集內被視為專家的使用者。 列出描述時，專家意見 (描述) 會移至 UX 頂端。 每個使用者可以指定自己的專家。 只有該使用者可以編輯 Expert 物件 (系統管理員和資產擁有者可以刪除 Expert 物件，但無法編輯它)。</td></tr>
<tr><td></td><td>expert</td><td>SecurityPrincipal</td><td></td></tr>

<tr><td>預覽 ("previews")</td><td></td><td></td><td>預覽包含資產的前 20 列資料的快照集。 預覽只對某些資產類型才有意義 (對資料表有意義，但對量值沒有意義)。</td></tr>
<tr><td></td><td>preview</td><td>object[]</td><td>代表資料行的物件陣列。  每個物件都有屬性與資料行的對應，根據的是資料列在該資料行中的值。</td></tr>

<tr><td>AccessInstruction ("accessInstructions")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>mimeType</td><td>字串</td><td>內容的 mime 類型。</td></tr>
<tr><td></td><td>內容</td><td>字串</td><td>如何取得這項資料資產的指示。 內容可能是一個 URL、電子郵件地址或一組指示。</td></tr>

<tr><td>TableDataProfile ("tableDataProfiles")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>numberOfRows</td></td><td>int</td><td>在此資料集的資料列數目。</td></tr>
<tr><td></td><td>size</td><td>long</td><td>以位元組為單位的資料集大小。  </td></tr>
<tr><td></td><td>schemaModifiedTime</td><td>字串</td><td>上次修改結構描述的時間。</td></tr>
<tr><td></td><td>dataModifiedTime</td><td>字串</td><td>上次修改此資料集 (新增、修改或刪除資料) 的時間</td></tr>

<tr><td>ColumnsDataProfile ("columnsDataProfiles")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>columns</td></td><td>ColumnDataProfile[]</td><td>資料行資料設定檔的陣列。</td></tr>

<tr><td>ColumnDataClassification ("columnDataClassifications")</td><td></td><td></td><td></td></tr>
<tr><td></td><td>columnName</td><td>字串</td><td>此分類所參考的資料行名稱。</td></tr>
<tr><td></td><td>分類</td><td>字串</td><td>此資料行中的資料分類。</td></tr>

<tr><td>文件 ("documentation")</td><td></td><td></td><td>指定的資產只能有一個相關聯的文件。</td></tr>
<tr><td></td><td>mimeType</td><td>字串</td><td>內容的 mime 類型。</td></tr>
<tr><td></td><td>內容</td><td>字串</td><td>文件內容。</td></tr>

</table>

### <a name="common-types"></a>常見的類型
常見類型可作為屬性的類型，而不是項目的類型。

<table>
<tr><td><b>一般類型</b></td><td><b>屬性</b></td><td><b>資料類型</b></td><td><b>註解</b></td></tr>
<tr><td>DataSourceInfo</td><td></td><td></td><td></td></tr>
<tr><td></td><td>sourceType</td><td>字串</td><td>描述資料來源的類型。  例如︰SQL Server、Oracle 資料庫等。  </td></tr>
<tr><td></td><td>objectType</td><td>字串</td><td>描述資料來源中的物件類型。 例如：SQL Server 的資料表、檢視。</td></tr>

<tr><td>DataSourceLocation</td><td></td><td></td><td></td></tr>
<tr><td></td><td>protocol</td><td>字串</td><td>必要。 描述用來與資料來源進行通訊的通訊協定。 例如：適用於 SQl Server 的 "tds"、適用於 Oracle 的 "oracle" 等等。如需目前支援的通訊協定清單，請參閱[資料來源參考規格 - DSL 結構](data-catalog-dsr.md)。</td></tr>
<tr><td></td><td>位址</td><td>字典<string, object></td><td>必要。 位址是一組專屬於通訊協定的資料，用來識別所參考的資料來源。 資料將範圍設定為特定通訊協定的位址，表示如果不知道通訊協定就沒有意義。</td></tr>
<tr><td></td><td>驗證</td><td>字串</td><td>選用。 用來與資料來源進行通訊的驗證配置。 例如：windows、oauth 等等。</td></tr>
<tr><td></td><td>connectionProperties</td><td>字典<string, object></td><td>選用。 如何連接資料來源的其他資訊。</td></tr>

<tr><td>SecurityPrincipal</td><td></td><td></td><td>後端不會在發佈期間對 AAD 驗證所提供的屬性。</td></tr>
<tr><td></td><td>upn</td><td>字串</td><td>使用者的唯一電子郵件地址。 如果未提供 objectId 或其位於 "lastRegisteredBy" 屬性的內容中則必須指定，否則為選擇性。</td></tr>
<tr><td></td><td>objectId</td><td>Guid</td><td>使用者或安全性群組 AAD 身分識別。 選用。 如果未提供 upn 則必須指定，否則為選擇性。</td></tr>
<tr><td></td><td>firstName</td><td>字串</td><td>使用者的名字 (用於顯示)。 選用。 只有在位於 "lastRegisteredBy" 屬性的內容中時才有效。 在為 "roles"、"permissions" 和 "experts" 提供安全性主體時則不能指定。</td></tr>
<tr><td></td><td>lastName</td><td>字串</td><td>使用者的姓氏 (用於顯示)。 選用。 只有在位於 "lastRegisteredBy" 屬性的內容中時才有效。 在為 "roles"、"permissions" 和 "experts" 提供安全性主體時則不能指定。</td></tr>

<tr><td>欄</td><td></td><td></td><td></td></tr>
<tr><td></td><td>name</td><td>字串</td><td>資料行或屬性的名稱。</td></tr>
<tr><td></td><td>type</td><td>字串</td><td>資料行或屬性的資料類型。 允許的類型取決於資產的資料 sourceType。  僅支援一部分類型。</td></tr>
<tr><td></td><td>maxLength</td><td>int</td><td>資料行或屬性允許的長度上限。 衍生自資料來源。 只適用於某些來源類型。</td></tr>
<tr><td></td><td>precision</td><td>byte</td><td>資料行或屬性的精確度。 衍生自資料來源。 只適用於某些來源類型。</td></tr>
<tr><td></td><td>isNullable</td><td>BOOLEAN</td><td>是否允許資料行有 Null 值。 衍生自資料來源。 只適用於某些來源類型。</td></tr>
<tr><td></td><td>expression</td><td>字串</td><td>如果值是導出資料行，此欄位會包含表示此值的運算式。 衍生自資料來源。 只適用於某些來源類型。</td></tr>

<tr><td>ColumnDataProfile</td><td></td><td></td><td></td></tr>
<tr><td></td><td>columnName </td><td>字串</td><td>資料行的名稱。</td></tr>
<tr><td></td><td>type </td><td>字串</td><td>資料行的類型。</td></tr>
<tr><td></td><td>Min </td><td>字串</td><td>在此資料集內的最小值。</td></tr>
<tr><td></td><td>max </td><td>字串</td><td>在此資料集內的最大值。</td></tr>
<tr><td></td><td>avg </td><td>double</td><td>在此資料集內的平均值。</td></tr>
<tr><td></td><td>stdev </td><td>double</td><td>此資料集的標準差。</td></tr>
<tr><td></td><td>nullCount </td><td>int</td><td>在此資料集內 null 值的計數。</td></tr>
<tr><td></td><td>distinctCount  </td><td>int</td><td>在此資料集內相異值的計數。</td></tr>


</table>

## <a name="asset-identity"></a>資產身分識別
Azure 資料目錄使用來自 DataSourceLocation "dsl" 屬性之「位址」屬性包的「通訊協定」和身分識別屬性，產生用來處理目錄中資產的資產身分識別。
例如，"tds" 通訊協定具有「伺服器」、「資料庫」、「結構描述」和「物件」等身分識別屬性。 通訊協定和身分識別屬性的組合可用來產生 SQL Server 資料表資產的身分識別。
Azure 資料目錄提供數個內建資料來源通訊協定，列在 [資料來源參考規格 - DSL 結構](data-catalog-dsr.md)中。
支援的通訊協定集可以利用程式設計方式擴充 (請參閱資料目錄 REST API 參考)。 目錄的系統管理員可以註冊自訂資料來源通訊協定。 下表描述註冊自訂通訊協定所需的屬性。

### <a name="custom-data-source-protocol-specification"></a>自訂資料來源通訊協定規格
<table>
<tr><td><b>類型</b></td><td><b>屬性</b></td><td><b>資料類型</b></td><td><b>註解</b></td></tr>

<tr><td>DataSourceProtocol</td><td></td><td></td><td></td></tr>
<tr><td></td><td>namespace</td><td>字串</td><td>通訊協定的命名空間。 命名空間長度必須介於 1 到 255 個字元，包含一或多個以點 (.) 分隔的非空白部分。 每個部分長度必須介於 1 到 255 個字元，以字母開頭並且只包含字母和數字。</td></tr>
<tr><td></td><td>name</td><td>字串</td><td>通訊協定的名稱。 名稱長度必須介於 1 到 255 個字元，以字母開頭並且只包含字母、數字和虛線 (-) 字元。</td></tr>
<tr><td></td><td>identityProperties</td><td>DataSourceProtocolIdentityProperty[]</td><td>身分識別屬性清單必須包含至少一個，但不超過 20 個屬性。 例如：「伺服器」、「資料庫」、「結構描述」、「物件」是 "tds" 通訊協定的身分識別屬性。</td></tr>
<tr><td></td><td>identitySets</td><td>DataSourceProtocolIdentitySet[]</td><td>身分識別集合的清單。 定義代表有效資產身分識別的身分識別屬性集合。 必須包含至少一個，但不超過 20 個集合。 例如：{「伺服器」、「資料庫」、「結構描述」和「物件」} 是 "tds" 通訊協定的身分識別集合，它會定義 Sql Server 資料表資產的身分識別。</td></tr>

<tr><td>DataSourceProtocolIdentityProperty</td><td></td><td></td><td></td></tr>
<tr><td></td><td>name</td><td>字串</td><td>屬性的名稱。 名稱長度必須介於 1 到 100 個字元，以字母開頭並且只能包含字母和數字。</td></tr>
<tr><td></td><td>type</td><td>字串</td><td>屬性的類型。 支援的值："bool"、"boolean"、"byte"、"guid"、"int"、"integer"、"long"、"string"、"url"</td></tr>
<tr><td></td><td>ignoreCase</td><td>布林</td><td>表示使用屬性的值時，是否應忽略大小寫。 只能對具有 "string" 類型的屬性指定。 預設值為 false。</td></tr>
<tr><td></td><td>urlPathSegmentsIgnoreCase</td><td>bool[]</td><td>指出是否應忽略 url 路徑之每個區段的大小寫。 只能對具有 "url" 類型的屬性指定。 預設值為 [false]。</td></tr>

<tr><td>DataSourceProtocolIdentitySet</td><td></td><td></td><td></td></tr>
<tr><td></td><td>name</td><td>字串</td><td>身分識別集合的名稱。</td></tr>
<tr><td></td><td>properties</td><td>string[]</td><td>身分識別屬性清單包含在這個身分識別集合中。 它不能包含重複項目。 每個身分識別集合所參考的屬性必須在通訊協定的 "identityProperties" 清單中定義。</td></tr>

</table>

## <a name="roles-and-authorization"></a>角色和授權
Microsoft Azure 資料目錄對資產和註解的 CRUD 作業提供授權功能。

## <a name="key-concepts"></a>重要概念
Azure 資料目錄使用兩種授權機制：

* 以角色為基礎的授權
* 以權限為基礎的授權

### <a name="roles"></a>角色
有 3 個角色：**系統管理員**、**擁有者**和**參與者**。  每個角色有其範圍和權限，於下表中摘要說明。

<table><tr><td><b>角色</b></td><td><b>範圍</b></td><td><b>權限</b></td></tr><tr><td>系統管理員</td><td>目錄 (目錄中的所有資產/註解)</td><td>Read Delete ViewRoles

ChangeOwnership ChangeVisibility ViewPermissions</td></tr><tr><td>擁有者</td><td>每個資產 (根項目)</td><td>Read Delete ViewRoles

ChangeOwnership ChangeVisibility ViewPermissions</td></tr><tr><td>參與者</td><td>每個個別的資產和註解</td><td>Read Update Delete ViewRoles 注意事項：如果撤銷 Contributor 在項目上的 Read 權限，則會撤鎖所有權限</td></tr></table>

> [!NOTE]
> **Read**、**Update**、**Delete**、**ViewRoles** 權限適用於任何項目 (資產或註解)，而 **TakeOwnership**、**ChangeOwnership**、**ChangeVisibility**、**ViewPermissions** 只適用於根資產。
> 
> **Delete** 權限適用於項目及其底下的任何子項目或單一項目。 例如，刪除資產時，也會刪除該資產的任何註解。
> 
> 

### <a name="permissions"></a>權限
權限是存取控制項目的清單。 每個存取控制項目指派一組權限給安全性主體。 權限只能在資產 (也就是根項目) 上指定，可套用至資產和任何子項目。

在 **Azure 資料目錄**預覽期間，僅支援權限清單中的 **Read** 權限，以便該情況下限制資產的可見性。

根據預設，任何已驗證的使用者對目錄中的任何項目都具有 **Read** 權限，除非權限中已限制只能看到某一組主體。

## <a name="rest-api"></a>REST API
**PUT** 和 **POST** 檢視項目要求可用於控制角色和權限：除了項目承載，還可指定兩個系統屬性 **roles** 和 **permissions**。

> [!NOTE]
> **permissions** 僅適用於根項目。
> 
> **擁有者** 角色僅適用於根項目。
> 
> 根據預設，在目錄中建立項目時，其 **參與者** 會設定為目前已驗證的使用者。 如果每個人都能更新項目，則第一次發佈項目時，應該在 **roles** 屬性中，將**參與者**設定為 &lt;Everyone&gt; 特殊安全性主體 (請參閱下列範例)。 **參與者**無法變更，而且在項目存留期間都維持不變 (即使**系統管理員**或**擁有者**都沒有權限變更**參與者**)。 明確設定**參與者**時，唯一支援的值是 &lt;Everyone&gt;：**參與者**只能是建立項目的使用者，或是 &lt;Everyone&gt;。
> 
> 

### <a name="examples"></a>範例
**發佈項目時將參與者設定為 &lt;Everyone&gt;。**
特殊安全性主體 &lt;Everyone&gt; 具有 objectId "00000000-0000-0000-0000-000000000201"。
  **POST** https://api.azuredatacatalog.com/catalogs/default/views/tables/?api-version=2016-03-30

> [!NOTE]
> 某些 HTTP 用戶端實作可能會自動重新發出要求，以從伺服器回應 302，但通常會從要求中刪除 Authorization 標頭。 因為需要有 Authorization 標頭才能對 Azure 資料目錄提出要求，所以您必須確定在對 Azure 資料目錄所指定的重新導向位置重新發出要求時，仍然會提供 Authorization 標頭。 下列範例程式碼使用 .NET HttpWebRequest 物件進行示範。
> 
> 

**內文**

    {
        "roles": [
            {
                "role": "Contributor",
                "members": [
                    {
                        "objectId": "00000000-0000-0000-0000-000000000201"
                    }
                ]
            }
        ]
    }

  **指派擁有者及限制現有根項目的可見度**：**PUT** https://api.azuredatacatalog.com/catalogs/default/views/tables/042297b0...1be45ecd462a?api-version=2016-03-30

    {
        "roles": [
            {
                "role": "Owner",
                "members": [
                    {
                        "objectId": "c4159539-846a-45af-bdfb-58efd3772b43",
                        "upn": "user1@contoso.com"
                    },
                    {
                        "objectId": "fdabd95b-7c56-47d6-a6ba-a7c5f264533f",
                        "upn": "user2@contoso.com"
                    }
                ]
            }
        ],
        "permissions": [
            {
                "principal": {
                    "objectId": "27b9a0eb-bb71-4297-9f1f-c462dab7192a",
                    "upn": "user3@contoso.com"
                },
                "rights": [
                    {
                        "right": "Read"
                    }
                ]
            },
            {
                "principal": {
                    "objectId": "4c8bc8ce-225c-4fcf-b09a-047030baab31",
                    "upn": "user4@contoso.com"
                },
                "rights": [
                    {
                        "right": "Read"
                    }
                ]
            }
        ]
    }

> [!NOTE]
> 在 PUT 中，不需要在內文中指定項目裝載：PUT 可以用來直接更新角色和 (或) 權限。
> 
> 

<!--Image references-->
[1]: ./media/data-catalog-developer-concepts/concept2.png
