---
title: 依國家/地區限制 Azure CDN 內容
description: 瞭解如何使用地區篩選功能，將每個國家/地區的存取限制為您的 Azure CDN 內容。
services: cdn
documentationcenter: ''
author: asudbring
ms.service: azure-cdn
ms.topic: how-to
ms.date: 01/16/2021
ms.author: allensu
ms.openlocfilehash: 8901dffb752409acd7fb08a2025bed9a4cc70132
ms.sourcegitcommit: fc23b4c625f0b26d14a5a6433e8b7b6fb42d868b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2021
ms.locfileid: "98539502"
---
# <a name="restrict-azure-cdn-content-by-countryregion"></a>依國家/地區限制 Azure CDN 內容

## <a name="overview"></a>概觀
當使用者要求您的內容時，會將內容提供給所有位置的使用者。 您可能會想要依國家/地區限制對內容的存取。 

使用 *地區篩選* 功能時，您可以在 CDN 端點上的特定路徑上建立規則。 您可以設定規則以允許或封鎖所選國家/地區中的內容。

> [!IMPORTANT]
> **來自 Microsoft 的標準 Azure CDN** 設定檔不支援以路徑為基礎的地區篩選。
> 

## <a name="standard-profiles"></a>標準設定檔

這些指示適用于 **來自 Akamai 的 AZURE Cdn 標準** ，以及來自 Verizon 設定檔的 **azure cdn 標準** 。

若為 **來自 Verizon 的 Azure CDN 進階** 設定檔，則必須使用 **管理** 入口網站來啟動地區篩選。 如需詳細資訊，請參閱[來自 Verizon 的 Azure CDN 進階設定檔](#azure-cdn-premium-from-verizon-profiles)。

### <a name="define-the-directory-path"></a>定義目錄路徑
若要存取地區篩選功能，請在入口網站內選取 CDN 端點，然後在左側功能表的 [設定] 下選取 [地區篩選]。 

![螢幕擷取畫面，顯示從端點的功能表中選取的地區篩選。](./media/cdn-filtering/cdn-geo-filtering-standard.png)

從 [路徑] 方塊中，指定要允許或拒絕使用者存取的位置相對路徑。 

您可藉由指定目錄路徑 (例如，/pictures/)，並使用正斜線 (/) 或選取特定資料夾，來對所有檔案套用地區篩選。 您也可以對單一檔案套用地區篩選 (例如，/pictures/city.png)。 允許多個規則。 輸入規則之後，會出現空白資料列，讓您輸入下一個規則。

例如，下列目錄路徑篩選條件全都有效：   
*/*                                 
*照片*     
*/Photos/Strasbourg/*     
/Photos/Strasbourg/city.png

### <a name="define-the-type-of-action"></a>定義動作的類型

從 [動作] 清單中，選取 [允許] 或 [封鎖]： 

- **允許**：只允許來自指定國家/地區的使用者存取從遞迴路徑要求的資產。

- **封鎖**：來自指定國家/地區的使用者將被拒絕存取從遞迴路徑要求的資產。 如果未設定該位置的其他國家/地區篩選選項，則會允許所有其他使用者進行存取。

例如，用於封鎖 /Photos/Strasbourg/ 路徑的地區篩選規則會篩選出下列檔案：     
*HTTP： \/ / \<endpoint> . azureedge.net/Photos/Strasbourg/1000.jpg* 
 *HTTP： \/ / \<endpoint> . azureedge.net/Photos/Strasbourg/Cathedral/1000.jpg*

### <a name="define-the-countriesregions"></a>定義國家/地區

從 [ **國家/地區代碼** ] 清單中，選取您要封鎖或允許路徑的國家/地區。 

當您完成選取國家/地區之後，請選取 [ **儲存** ] 以啟用新的地區篩選規則。 

![螢幕擷取畫面顯示用來封鎖或允許國家或地區的國家/地區代碼。](./media/cdn-filtering/cdn-geo-filtering-rules.png)

### <a name="clean-up-resources"></a>清除資源

若要刪除規則，請從 [地區篩選] 頁面上的清單中選取規則，然後選擇 [刪除]。

## <a name="azure-cdn-premium-from-verizon-profiles"></a>來自 Verizon 的 Azure CDN 進階設定檔

針對 **來自 Verizon 設定檔的 AZURE CDN Premium，用** 來建立地區篩選規則的使用者介面會不同：

1. 在 Azure CDN 設定檔的頂端功能表中，選取 [管理]。

2. 在 Verizon 入口網站中，選取 [HTTP 大型]，然後選取 [國家 (地區) 篩選]。

    :::image type="content" source="./media/cdn-filtering/cdn-geo-filtering-premium.png" alt-text="螢幕擷取畫面顯示如何在 Azure CDN 中選取國家/地區篩選" border="true":::
  
3. 選取 [新增國家 (地區) 篩選]。

4. 在 [ **步驟1：**] 中，輸入目錄路徑。 選取 [ **封鎖** ] 或 [ **新增**]，然後選取 **[下一步]**。

    > [!IMPORTANT]
    > 端點名稱必須在路徑中。  範例： **/myendpoint8675/myfolder**。  將 **myendpoint8675** 取代為您端點的名稱。
    > 
    
5. 在 **步驟 2** 中，從清單中選取一或多個國家/地區。 選取 **[完成]** 以啟動規則。 
    
    新的規則便會出現在 [國家 (地區)篩選] 頁面的資料表中。
    
    :::image type="content" source="./media/cdn-filtering/cdn-geo-filtering-premium-rules.png" alt-text="螢幕擷取畫面：顯示規則在國家（地區）篩選中的顯示位置。" border="true":::
 
### <a name="clean-up-resources"></a>清除資源
在 [國家/地區篩選規則] 資料表中，選取規則旁邊的 [刪除] 圖示以刪除它，或選取 [編輯] 圖示加以修改。

## <a name="considerations"></a>考量
* 您地區篩選設定的變更不會立即生效：
   * 若為 **來自 Microsoft 的標準 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
   * 若為 **來自 Akamai 的標準 Azure CDN** 設定檔，通常會在一分鐘內完成傳播。 
   * 若為 **來自 Verizon 的標準 Azure CDN** 和 **來自 Verizon 的進階 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
 
* 這項功能不支援萬用字元 (例如 * ) 。

* 會將相對路徑相關聯的地區篩選組態，遞迴地套用到該路徑。

* 只能對相同的相對路徑套用一個規則。 也就是說，您無法建立指向相同相對路徑的多個國家/地區篩選。 不過，因為國家/地區篩選是遞迴的，所以資料夾可以有多個國家/地區篩選。 換句話說，您可以將不同的國家/地區篩選指派給先前設定之資料夾的子資料夾。

* 地區篩選功能使用國家（地區）代碼來定義允許或封鎖受保護目錄之要求的國家/地區。 雖然 Akamai 和 Verizon 設定檔可支援大部分的相同國碼 (地區碼)，但兩者有一些差異。 如需詳細資訊，請參閱 [AZURE CDN 國家/地區代碼](/previous-versions/azure/mt761717(v=azure.100))。 

