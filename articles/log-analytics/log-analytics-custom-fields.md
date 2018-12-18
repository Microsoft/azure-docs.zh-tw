---
title: Azure Log Analytics 中的自訂欄位 | Microsoft Docs
description: Log Analytics 的自訂欄位功能，可讓您從新增到所收集記錄之屬性的 Log Analytics 記錄建立自己的可搜尋欄位。  本文說明用來建立自訂欄位的程序，並透過範例事件提供詳細的逐步解說。
services: log-analytics
documentationcenter: ''
author: bwren
manager: jwhit
editor: tysonn
ms.assetid: 31572b51-6b57-4945-8208-ecfc3b5304fc
ms.service: log-analytics
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/23/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 164b071a47c13f7c5586bd287adc41bea54a1198
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37129612"
---
# <a name="custom-fields-in-log-analytics"></a>Log Analytics 中的自訂欄位
Log Analytics 的**自訂欄位**功能，可讓您透過新增自己的可搜尋欄位來擴充 Log Analytics 中的現有記錄。  自訂欄位會自動填入擷取自同一筆記錄中其他屬性的資料。

![自訂欄位概觀](media/log-analytics-custom-fields/overview.png)

例如，以下範例記錄在事件描述中埋藏了有用的資料。  擷取這項資料並將其分成不同屬性，將可讓您對資料進行排序和篩選等動作。

![記錄檔搜尋按鈕](media/log-analytics-custom-fields/sample-extract.png)

> [!NOTE]
> 在預覽版中，工作區限制只能有 100 個自訂欄位。  這項功能正式上市時，此限制數量將會擴大。
> 
> 

## <a name="creating-a-custom-field"></a>建立自訂欄位
當您在建立自訂欄位時，Log Analytics 必須了解要用來填入其值的資料。  它會使用來自 Microsoft Research 稱為 FlashExtract 的技術快速識別此資料。  Log Analytics 不會要求您提供明確的指示，而是會了解您想要從提供的範例中擷取的資料。

下列各節提供用於建立自訂欄位的程序。  本文最後會逐步解說範例擷取作業。

> [!NOTE]
> 將符合指定準則的記錄新增到 Log Analytics 時便會填入自訂欄位，因此，它只會出現在建立自訂欄位之後所收集的記錄上。  建立自訂欄位時便已存在於資料存放區的記錄不會新增自訂欄位。
> 

### <a name="step-1--identify-records-that-will-have-the-custom-field"></a>步驟 1 – 識別將會具有自訂欄位的記錄
第一個步驟是識別會取得自訂欄位的記錄。  一開始您可以先進行 [標準的記錄檔搜尋](log-analytics-log-searches.md) ，然後再選取要做為 Log Analytics 將從中了解之模型的記錄。  當您指定您要將資料擷取到自訂欄位時，便會開啟 [欄位擷取精靈]  供您驗證及精簡準則。

1. 移至 [記錄檔搜尋]  ，然後使用 [查詢來擷取將會具有自訂欄位的記錄](log-analytics-log-searches.md) 。
2. 選取 Log Analytics 將用來做為模型以擷取資料來填入自訂欄位的記錄。  您會識別您想要從這筆記錄擷取的資料，Log Analytics 會使用這項資訊來判斷要為所有類似記錄填入自訂欄位的邏輯。
3. 按一下記錄的任何文字屬性左邊的按鈕，然後選取 [擷取欄位來源] 。
4. [欄位擷取精靈] 會隨即開啟，而您選取的記錄會顯示在 [主要範例] 資料行。  這些記錄的自訂欄位將會以所選屬性中的相同值來定義。  
5. 如果已選取的項目未完全符合您想要選取的項目，請選取其他欄位以縮小準則範圍。  若要變更準則的欄位值，您必須先取消，再選取符合您所需準則的不同記錄。

### <a name="step-2---perform-initial-extract"></a>步驟 2 - 執行初始擷取。
一旦您識別出將會具有自訂欄位的記錄，您就已識別您想要擷取的資料。  Log Analytics 會使用這項資訊來識別類似記錄中的類似模式。  在這之後的步驟中，您將可以驗證結果，並提供更進一步的詳細資料以供 Log Analytics 用於其分析中。

1. 在範例記錄中醒目提示您想要填入自訂欄位的文字。  然後您會看到一個對話方塊，供您提供欄位名稱和執行初始擷取。  字元 **\__CF** 會自動附加上去。
2. 按一下 [擷取]  以對收集的記錄進行分析。  
3. [摘要] 和 [搜尋結果] 區段會顯示擷取結果，以供您檢查其正確性。   會顯示用來識別記錄的準則，以及每個所識別之資料值的計數。   會提供符合準則之記錄的詳細清單。

### <a name="step-3--verify-accuracy-of-the-extract-and-create-custom-field"></a>步驟 3 – 驗證擷取的正確性並建立自訂欄位
一旦您執行過初始擷取，Log Analytics 就會根據收集到的資料顯示其結果。  如果結果看起來是正確的，您就可以建立自訂欄位，無須再進行其他工作。  如果不正確，則可以精簡結果以便讓 Log Analytics 提升其邏輯。

1. 如果初始擷取中有任何值不正確，則請按一下錯誤記錄旁的 [編輯] 圖示，然後選取 [修改此醒目提示] 以修改選取項目。
2. 項目會複製到 [主要範例] 底下的 [其他範例] 區段。  您可以在此調整醒目提示來協助 Log Analytics 了解其應該選取的項目。
3. 按一下 [擷取]  ，使用這項新資訊來評估所有現有記錄。  根據這個新情報，記錄的結果可能會有所修改，而不同於您剛才修改的記錄。
4. 繼續加入修正，直到擷取中的所有記錄正確識別要填入新自訂欄位的資料。
5. 當您滿意結果時，按一下 [儲存擷取]  。  自訂欄位現已定義完成，但還不會新增到任何記錄中。
6. 請等候符合指定準則的新記錄收集完成，然後再次執行記錄檔搜尋。 新的記錄應該會有自訂欄位。
7. 和任何其他記錄屬性一樣地使用自訂欄位。  您可以使用它來彙總與群組資料，甚至用它來產生新的見解。

## <a name="viewing-custom-fields"></a>檢視自訂欄位
您可以在 Azure 入口網站中，從 Log Analytics 工作區的 [進階設定] 功能表檢視管理群組中的所有自訂欄位清單。  依序選取 [資料] 和 [自訂欄位]，可取得工作區中所有自訂欄位的清單。  

![自訂欄位](media/log-analytics-custom-fields/list.png)

## <a name="removing-a-custom-field"></a>移除自訂欄位
有兩種方式可移除自訂欄位。  第一個方式是在檢視如上所述的完整清單時，每個欄位所具有的 [移除]  選項。  另一個方式是擷取一筆記錄，然後按一下欄位左邊的按鈕。  功能表中會有可供移除自訂欄位的選項。

## <a name="sample-walkthrough"></a>範例逐步解說
下面的章節會逐步解說建立自訂欄位的完整範例。  這個範例會擷取 Windows 事件中指出變更狀態之服務的服務名稱。  這個動作必須仰賴服務控制管理員在 Windows 電腦的系統記錄檔中建立的事件。  如果您想要跟隨此範例，您必須 [收集系統記錄檔的資訊事件](log-analytics-data-sources-windows-events.md)。

我們輸入下列查詢，以從服務控制管理員傳回事件識別碼為 7036，且為指出服務正在啟動或停止之事件的所有事件。

![查詢](media/log-analytics-custom-fields/query.png)

然後，我們選取任何事件識別碼為 7036 的記錄。

![來源記錄](media/log-analytics-custom-fields/source-record.png)

我們想要讓該服務名稱出現在 [RenderedDescription]  屬性，並選取這個屬性旁的按鈕。

![擷取欄位](media/log-analytics-custom-fields/extract-fields.png)

[欄位擷取精靈] 會隨即開啟，而且 [主要範例] 資料行中已選取 [EventLog] 和 [EventID] 欄位。  這表示將會為系統記錄檔中事件識別碼為 7036 的事件定義自訂欄位。  這樣就可以了，所以我們不需要再選取其他欄位。

![主要範例](media/log-analytics-custom-fields/main-example.png)

我們醒目提示 [RenderedDescription] 屬性中的服務名稱，然後使用 [Service] 來識別服務名稱。  自訂欄位將會稱為 **Service_CF**。

![欄位標題](media/log-analytics-custom-fields/field-title.png)

我們看到某些記錄已正確識別服務名稱，但某些記錄則未如此。   [搜尋結果] 顯示 **WMI Performance Adapter** 有部分名稱未能選取到。  [摘要] 顯示四筆具有 **DPRMA** 服務的記錄錯誤地包含了額外的文字，而有兩筆記錄識別了 **Modules Installer** 而不是 **Windows Modules Installer**。  

![搜尋結果](media/log-analytics-custom-fields/search-results-01.png)

我們先處理 **WMI Performance Adapter** 記錄。  我們按一下它的編輯圖示，然後 [修改此醒目提示] 。  

![修改醒目提示](media/log-analytics-custom-fields/modify-highlight.png)

我們增加醒目提示範圍來包含 **WMI** 一字，然後重新執行擷取。  

![其他範例](media/log-analytics-custom-fields/additional-example-01.png)

我們可以看到 **WMI Performance Adapter** 的項目已修正，而且 Log Analytics 也已使用該資訊來更正 **Windows Module Installer** 的記錄。  但我們可以看到 [摘要] 區段中 **DPMRA** 仍未被正確識別。

![搜尋結果](media/log-analytics-custom-fields/search-results-02.png)

我們捲動到具有 DPMRA 服務的記錄，並使用相同的程序來更正該記錄。

![其他範例](media/log-analytics-custom-fields/additional-example-02.png)

 當我們執行擷取時，我們可以看到所有結果現在都已正確。

![搜尋結果](media/log-analytics-custom-fields/search-results-03.png)

我們可以看到 **Service_CF** 已建立但尚未新增到任何記錄中。

![初始計數](media/log-analytics-custom-fields/initial-count.png)

過了一會兒之後，新事件已收集完成，我們可以看到 [Service_CF] 欄位現在會新增到符合準則的記錄中。

![最終結果](media/log-analytics-custom-fields/final-results.png)

現在我們可以和任何其他記錄屬性一樣地使用自訂欄位。  為了說明這一點，我們建立以新的 [Service_CF] 欄位來群組的查詢，以檢查哪些服務最常使用。

![以查詢分組](media/log-analytics-custom-fields/query-group.png)

## <a name="next-steps"></a>後續步驟
* 了解 [記錄搜尋](log-analytics-log-searches.md) ，以使用自訂欄位作為準則來建立查詢。
* 監視可利用自訂欄位來剖析的[自訂記錄檔](log-analytics-data-sources-custom-logs.md)。

