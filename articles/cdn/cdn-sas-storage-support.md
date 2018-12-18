---
title: 搭配 SAS 使用 Azure CDN | Microsoft Docs
description: Azure CDN 支援使用「共用存取簽章」(SAS) 來授與有限的私人儲存體容器存取權。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/21/2018
ms.author: magattus
ms.openlocfilehash: 7180e51a6ac1392e4a3f072097b1aeef3648c605
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093284"
---
# <a name="using-azure-cdn-with-sas"></a>搭配 SAS 使用 Azure CDN

當您設定儲存體帳戶供「Azure 內容傳遞網路」(CDN) 用來快取內容時，根據預設，所有知道您儲存體容器 URL 的人都能存取您已上傳的檔案。 若要保護您儲存體帳戶中的檔案，您可以將儲存體容器的存取權從公用改設定成私用。 不過，如果您這樣做，所有人都將無法存取您的檔案。 

如果您想要授與有限的私人儲存體容器存取權，便可以使用 Azure 儲存體帳戶的「共用存取簽章」(SAS) 功能。 SAS 是一個 URI，可在不公開您帳戶金鑰的情況下，授與對「Azure 儲存體」資源的有限存取權限。 針對您不信任其使用您儲存體帳戶金鑰但又想要將特定儲存體帳戶資源存取權委派給它們的用戶端，您可以將 SAS 提供給它們。 藉由將共用存取簽章 URI 提供給這些用戶端，您會授與它們一段指定時間的資源存取權。
 
使用 SAS 時，您可以定義各種 Blob 存取參數，例如開始時間和到期時間、權限 (讀取/寫入) 及 IP 範圍。 本文說明如何將 SAS 與 Azure CDN 搭配使用。 如需有關 SAS 的詳細資訊 (包括如何建立 SAS 及其參數選項)，請參閱[使用共用存取簽章 (SAS)](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)。

## <a name="setting-up-azure-cdn-to-work-with-storage-sas"></a>設定 Azure CDN 以與儲存體 SAS 搭配運作
以下是搭配 Azure CDN 使用 SAS 時的三個建議選項。 所有選項皆假設您已經建立一個可運作的 SAS (請參閱先決條件)。 
 
### <a name="prerequisites"></a>必要條件
若要開始，請建立一個儲存體帳戶，然後為您的資產生 SAS。 您可以產生兩種類型的預存存取簽章：服務 SAS 或帳戶 SAS。 如需詳細資訊，請參閱[共用存取簽章的類型](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1#types-of-shared-access-signatures)。

產生 SAS 權杖之後，您可以將 `?sv=<SAS token>` 附加至您的 URL 來存取 Blob 儲存體檔案。 此 URL 的格式如下： 

`https://<account name>.blob.core.windows.net/<container>/<file>?sv=<SAS token>`
 
例如︰
 ```
https://democdnstorage1.blob.core.windows.net/container1/demo.jpg?sv=2017-07-29&ss=b&srt=co&sp=r&se=2038-01-02T21:30:49Z&st=2018-01-02T13:30:49Z&spr=https&sig=QehoetQFWUEd1lhU5iOMGrHBmE727xYAbKJl5ohSiWI%3D
```

如需有關設定參數的詳細資訊，請參閱 [SAS 參數考量](#sas-parameter-considerations)和[共用存取簽章參數](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1#shared-access-signature-parameters)。

![CDN SAS 設定](./media/cdn-sas-storage-support/cdn-sas-settings.png)

### <a name="option-1-using-sas-with-pass-through-to-blob-storage-from-azure-cdn"></a>選項 1：使用具有從 Azure CDN 傳遞至 Blob 儲存體的 SAS

此選項是最簡單的選項，僅使用從 Azure CDN 傳遞至原始伺服器的單一 SAS 權杖。 **Azure CDN Standard from Verizon** 和 **Azure CDN Standard from Akamai** 設定檔支援此選項。 
 
1. 選取一個端點、選取 [快取規則]，然後從 [查詢字串快取] 清單中，選取 [快取所有不重複的 URL]。

    ![CDN 快取規則](./media/cdn-sas-storage-support/cdn-caching-rules.png)

2. 在儲存體帳戶上設定 SAS 之後，您必須搭配 CDN 端點和原始伺服器 URL 來使用 SAS 權杖以存取檔案。 
   
   產生的 CDN 端點 URL 會具有以下格式：`https://<endpoint hostname>.azureedge.net/<container>/<file>?sv=<SAS token>`

   例如︰   
   ```
   https://demoendpoint.azureedge.net/container1/demo.jpg/?sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   
3. 使用快取規則或在原始伺服器新增 `Cache-Control` 標頭來微調快取持續時間。 由於 Azure CDN 會將 SAS 權杖視為純查詢字串，因此最佳做法是，您應該設定一個在 SAS 到期時間或此時間之前到期的快取持續時間。 否則，如果檔案的快取持續時間比 SAS 的有效期長，便可能在過了 SAS 到期時間之後，從 Azure CDN 原始伺服器存取該檔案。 如果發生這種情況，而您想要讓已快取的檔案變成無法存取，就必須對該檔案執行清除作業，以將它從快取中清除。 如需有關在 Azure CDN 上設定快取持續時間的資訊，請參閱[使用快取規則來控制 Azure CDN 快取行為](cdn-caching-rules.md)。

### <a name="option-2-hidden-cdn-sas-token-using-a-rewrite-rule"></a>選項 2：使用重寫規則的隱藏式 CDN SAS 權杖
 
此選項僅適用於**來自 Verizon 的 Azure 進階 CDN**設定檔。 使用此選項時，您可以保護原始伺服器的 Blob 儲存體。 如果您不需要特定的檔案存取限制，但想要防止使用者直接存取儲存體原始伺服器來縮短 Azure CDN 卸載時間，便可以使用此選項。 存取原始伺服器指定容器中檔案的任何人都必須有使用者未知的 SAS 權杖。 不過，由於 URL 重寫規則，CDN 端點上不需要 SAS 權杖。
 
1. 使用[規則引擎](cdn-rules-engine.md)來建立 URL 重寫規則。 新規則大約需要 10 分鐘的時間來進行傳播。

   ![CDN [管理] 按鈕](./media/cdn-sas-storage-support/cdn-manage-btn.png)

   ![CDN [規則引擎] 按鈕](./media/cdn-sas-storage-support/cdn-rules-engine-btn.png)

   下列範例 URL 重寫規則使用了規則運算式模式，其中包含一個擷取群組和一個名為 *storagedemo* 的端點：
   
   來源：   
   `(\/container1\/.*)`
   
   目的地：   
   ```
   $1?sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   ![CDN URL 重寫規則 - 左](./media/cdn-sas-storage-support/cdn-url-rewrite-rule.png)
   ![CDN URL 重寫規則 - 右](./media/cdn-sas-storage-support/cdn-url-rewrite-rule-option-2.png)

2. 新規則生效後，任何人都可以存取 CDN 端點上指定容器中的檔案，不論他們是否在 URL 中使用 SAS 權杖都行。 格式如下：`https://<endpoint hostname>.azureedge.net/<container>/<file>`
 
   例如︰   
   `https://demoendpoint.azureedge.net/container1/demo.jpg`
       

3. 使用快取規則或在原始伺服器新增 `Cache-Control` 標頭來微調快取持續時間。 由於 Azure CDN 會將 SAS 權杖視為純查詢字串，因此最佳做法是，您應該設定一個在 SAS 到期時間或此時間之前到期的快取持續時間。 否則，如果檔案的快取持續時間比 SAS 的有效期長，便可能在過了 SAS 到期時間之後，從 Azure CDN 原始伺服器存取該檔案。 如果發生這種情況，而您想要讓已快取的檔案變成無法存取，就必須對該檔案執行清除作業，以將它從快取中清除。 如需有關在 Azure CDN 上設定快取持續時間的資訊，請參閱[使用快取規則來控制 Azure CDN 快取行為](cdn-caching-rules.md)。

### <a name="option-3-using-cdn-security-token-authentication-with-a-rewrite-rule"></a>選項 3：搭配重寫規則使用 CDN 安性權杖驗證

若要使用 Azure CDN 安全性權杖驗證，您必須具有 **來自 Verizon 的 Azure 進階 CDN** 設定檔。 此選項是最安全且可自訂的選項。 用戶端存取權取決於您在安全性權杖上設定的安全性參數。 建立並設定安全性權杖後，所有 CDN 端點 URL 上都需要此權杖。 不過，由於 URL 重寫規則，CDN 端點上不需要 SAS 權杖。 如果 SAS 權杖之後變成無效，Azure CDN 將無法再重新驗證來自原始伺服器的內容。

1. [建立 Azure CDN 安全性權杖](https://docs.microsoft.com/azure/cdn/cdn-token-auth#setting-up-token-authentication)，然後藉由您的使用者可存取檔案所在 CDN 端點和路徑的規則引擎來啟用權杖。

   安全性權杖端點 URL 的格式如下：   
   `https://<endpoint hostname>.azureedge.net/<container>/<file>?<security_token>`
 
   例如︰   
   ```
   https://demoendpoint.azureedge.net/container1/demo.jpg?a4fbc3710fd3449a7c99986bkquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
       
   安全性權杖驗證的參數選項與 SAS 權杖的參數選項不同。 如果您在建立安全性權杖時選擇使用到期時間，請將其設定為與 SAS 權杖到期時間相同的值。 這麼做可確保到期時間是可預測的。 
 
2. 使用[規則引擎](cdn-rules-engine.md)來建立 URL 重寫規則，以啟用對容器中所有 Blob 的 SAS 權杖存取權。 新規則大約需要 10 分鐘的時間來進行傳播。

   下列範例 URL 重寫規則使用了規則運算式模式，其中包含一個擷取群組和一個名為 *storagedemo* 的端點：
   
   來源：   
   `(\/container1\/.*)`
   
   目的地：   
   ```
   $1&sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   ![CDN URL 重寫規則 - 左](./media/cdn-sas-storage-support/cdn-url-rewrite-rule.png)
   ![CDN URL 重寫規則 - 右](./media/cdn-sas-storage-support/cdn-url-rewrite-rule-option-3.png)

3. 如果您更新 SAS，請務必將 Url 重寫規則更新成使用新的 SAS 權杖。 

## <a name="sas-parameter-considerations"></a>SAS 參數考量

由於 Azure CDN 看不到 SAS 參數，因此 Azure CDN 無法根據這些參數變更其傳遞行為。 所定義的參數限制僅適用於 Azure CDN 對原始伺服器提出的要求，而不適用於用戶端對 Azure CDN 提出的要求。 當您設定 SAS 參數時，請務必考量這項區別。 如果需要這些進階功能且您使用[選項 3](#option-3-using-cdn-security-token-authentication-with-a-rewrite-rule)，請在 Azure CDN 安全性權杖上設定適當的限制。

| SAS 參數名稱 | 說明 |
| --- | --- |
| Start | Azure CDN 可以開始存取 Blob 檔案的時間。 由於時鐘誤差 (當時鐘訊號針對不同元件在不同時間抵達時) 的緣故，因此如果您想要讓資產立即可供使用，請選擇提早 15 分鐘的時間。 |
| End | 之後 Azure CDN 即無法再存取 Blob 檔案的時間。 Azure CDN 上先前快取的檔案仍然可供存取。 若要控制檔案到期時間，請在 Azure CDN 安全性權杖上設定適當的到期時間或清除資產。 |
| 允許的 IP 位址 | 選用。 如果您使用**來自 Verizon 的 Azure CDN**，您可以將此參數設定為[來自 Verizon 的 Azure CDN Edge Server IP 範圍](https://msdn.microsoft.com/library/mt757330.aspx) \(英文\) 中定義的範圍。 如果您使用**來自 Akamai 的 Azure CDN**，則無法設定 IP 範圍參數，因為 IP 位址並非靜態。|
| 允許的通訊協定 | 針對使用帳戶 SAS 來提出之要求允許的通訊協定。 建議使用 HTTPS 設定。|

## <a name="next-steps"></a>後續步驟

如需 SAS 的詳細資訊，請參閱下列文章：
- [使用共用存取簽章 (SAS)](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)
- [共用存取簽章，第 2 部分：透過 Blob 儲存體來建立與使用 SAS](https://docs.microsoft.com/azure/storage/blobs/storage-dotnet-shared-access-signature-part-2)

如需設定權杖驗證的詳細資訊，請參閱[使用權杖驗證來保護 Azure 內容傳遞網路資產](https://docs.microsoft.com/azure/cdn/cdn-token-auth)。
