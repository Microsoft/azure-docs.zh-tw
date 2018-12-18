---
title: Azure 監視器中的自訂計量
description: 了解 Azure 監視器中的自訂計量，以及如何予以模型化。
author: ancav
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ancav
ms.component: metrics
ms.openlocfilehash: c136772e27dab014c22234f1ef1d2baddd2ffe58
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978075"
---
# <a name="custom-metrics-in-azure-monitor"></a>Azure 監視器中的自訂計量

在 Azure 中部署資源與應用程式時，您會想要開始收集遙測，以取得其效能與健康情況的深入解析。 Azure 會在您部署資源時，提供一些現成的計量。 這些稱為標準或平台計量。 不過，這些計量本質上有限制。 您可能想要收集一些自訂的效能指標或特定的商務計量，以提供更深入的見解。
您可以透過應用程式遙測、在 Azure 資源上執行的代理程式，或甚至是由外到內的監視系統，收集這些「自訂」計量，然後直接提交給 Azure 監視器。 發佈至 Azure 監視器之後，您可以和 Azure 所發出的標準計量並排瀏覽、查詢 Azure 資源與應用程式的自訂計量，以及對其發出警示。

## <a name="send-custom-metrics"></a>傳送自訂計量
自訂計量可以透過各種不同的方法，傳送至 Azure 監視器。
- 使用 Application Insights SDK 檢測您的應用程式，並將自訂遙測傳送至 Azure 監視器 
- 將 Windows 診斷擴充功能安裝在 [Azure VM](metrics-store-custom-guestos-resource-manager-vm.md)、[虛擬機器擴展集](metrics-store-custom-guestos-resource-manager-vmss.md)、[傳統 VM](metrics-store-custom-guestos-classic-vm.md) 或[傳統雲端服務](metrics-store-custom-guestos-classic-cloud-service.md)，並將效能計數器傳送至 Azure 監視器 
- 將 [InfluxDB Telegraf 代理程式](metrics-store-custom-linux-telegraf.md)安裝在 Azure Linux VM，並使用 Azure 監視器輸出外掛程式傳送計量
- 將自訂計量[直接傳送至 Azure 監視器 REST API](metrics-store-custom-rest-api.md) https://<azureregion>.monitoring.azure.com/<AzureResourceID>/metrics

當您將自訂計量傳送至 Azure 監視器時，報告的每個資料點 (或值) 都必須包含下列資訊：

### <a name="authentication"></a>驗證
若要將自訂計量提交至 Azure 監視器，提交計量的實體在要求的 "Bearer" 標頭中，需要具備有效的 Azure Active Directory 權杖。 支援取得有效持有人權杖的方式有好幾種：
1. [MSI (受控服務識別)](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)：對 Azure 資源本身 (例如 VM) 提供身分識別。 MSI 的設計可提供資源執行特定作業的權限；例如，允許資源發出自身的相關計量。 還可以在其他資源上授與資源 (或其 MSI) 的「監視計量發行者」權限，藉以讓 MSI 也能發出其他資源的計量。
2. [AAD 服務主體](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)：以下的案例是，將權限指派給 AAD 應用程式 (服務)，以發出 Azure 資源的相關計量。
若要驗證要求，Azure 監視器會使用 AAD 公開金鑰驗證應用程式權杖。 現有的「監視計量發行者」角色已有 Azure 入口網站中提供的這個權限。 可根據要為其發出自訂計量的資源，在所需的範圍 (訂用帳戶、資源群組或特定資源) 將「監視計量發行者」角色指定給服務主體。

> [!NOTE]
> 當要求 AAD 權杖以發出自訂計量時，請確定要求權杖的受眾/資源為 https://monitoring.azure.com/ (請務必包含尾端的斜線 '/')

### <a name="subject"></a>主體
這個屬性會擷取所報告自訂計量的 Azure 資源識別碼。 此資訊會編碼在所進行 API 呼叫的 URL 中。 每個 API 都只能為單一的 Azure 資源提交計量值。

> [!NOTE]
> 您無法針對資源群組或訂用帳戶的資源識別碼發出自訂計量。
>
>

### <a name="region"></a>區域
這個屬性會擷取您為其發出計量的資源部署所在的 Azure 區域。 對其發出計量的 Azure 監視器端點所在區域必須和資源的部署區域相同。 例如，部署在美國西部的 VM，其自訂計量必須傳送至美國西部的區域 Azure 監視器端點。 區域資訊也是編碼在 API 呼叫的 URL 中。

> [!NOTE]
> 自訂計量在公開預覽期間，只能在 Azure 區域的子集中使用。 支援的區域清單記載於本文稍後的章節。
>
>

### <a name="timestamp"></a>時間戳記
傳送至 Azure 監視器每個資料點都必須以時間戳記標記。 此時間戳記會擷取該計量值的測量/收集日期時間。 Azure 監視器接受時間戳記為過去 20 分鐘內和未來 5 分鐘內的計量資料。

### <a name="namespace"></a>命名空間
命名空間是將類似計量分類或分組的方法。 命名空間可讓您將收集不同的見解或效能指標的計量群組隔離。 例如，您可能有稱為 *ContosoMemoryMetrics* 的命名空間，用於可剖析應用程式的追蹤記憶體使用量計量，以及另一個稱為 *ContosoAppTransaction* 的命名空間，追蹤應用程式中與使用者交易相關的所有計量資料。

### <a name="name"></a>名稱
要報告的計量名稱。 通常名稱的描述就足以協助識別要測量的項目。 例如，會測量指定 VM 上所使用記憶體位元組數目的計量，其計量名稱會類似「使用中的記憶體位元組」。

### <a name="dimension-keys"></a>維度索引鍵
維度是索引鍵/值組，可協助描述所收集計量的其他相關特性。 其他特性能夠讓您收集計量的更多相關資訊，允許取得更深入的見解。 例如，「使用中的記憶體位元組」計量的維度索引鍵可能稱為「處理序」，可擷取 VM 上每個處理序正在使用的記憶體位元組數目。 這可讓您篩選計量，以查看正在使用多少個記憶體特定處理序，或識別記憶體使用量的前 5 名處理序。
每個自訂計量最多可以有 10 個維度。

### <a name="dimension-values"></a>維度值
當報告計量資料點時，要報告計量上的每個維度索引鍵都會有一個對應的維度值。 例如，如果您想要報告 VM 上由 ContosoApp 使用的記憶體：

* 計量名稱會是「使用中的記憶體位元組」
* 維度索引鍵會是「處理序」
* 維度值會是 *ContosoApp.exe*

當發佈計量值時，每個維度索引鍵只能指定單一維度值。 如果您會對 VM 上的多個處理序收集同一個記憶體使用量，您可以報告該時間戳記的多個計量值。 每個計量值都會為處理序維度索引鍵指定不同的維度值。

### <a name="metric-values"></a>計量值
Azure 監視器會儲存一分鐘資料粒度間隔內的所有計量。 我們了解在指定的分鐘時段內，計量可能必須取樣多次 (例如， CPU 使用率)，或測量許多離散事件 (例如， 登入交易延遲)。 若要限制您在 Azure 監視器中必須發出和支付未經處理的值數目，您可以在本機預先彙總，然後再將值發出：

* 最小值︰在該分鐘期間的所有樣本/測量中觀察到的最小值
* 最大值︰在該分鐘期間的所有樣本/測量中觀察到的最大值
* 總和︰在該分鐘期間的所有樣本/測量中所有觀察值的總和
* 計數：在該分鐘期間的樣本/測量數目

例如，如果在指定的分鐘時段內，您的應用程式有 4 個登入交易，所產生的測量延遲分別如下：

|交易 1|交易 2|交易 3|交易 4|
|---|---|---|---|
|7 毫秒|4 毫秒|13 毫秒|16 毫秒|
|

對 Azure 監視器產生的計量發行集如下：
* 最小值：4
* 最大值：16
* 總和：40
* 計數：4

如果您的應用程式無法在本機預先彙總，而必須在收集後立即發出每個離散樣本或事件，您可以發出未經處理的測量值。
例如，應用程式上每次發生登入交易時，您發佈到 Azure 監視器的計量只有單一測量結果。 因此，對於花費 12 毫秒的登入交易，其計量發行集如下：
* 最小值：12
* 最大值：12
* 總和：12
* 計數：1

此處理序可讓您在指定的分鐘時段內，為相同的計量 + 維度組合發出多個值。 Azure 監視器會接收指定的分鐘內發出的所有未經處理的值，然後再彙總在一起。

### <a name="sample-custom-metric-publication"></a>範例自訂計量發行集
在下列範例中，您會在虛擬機器的計量命名空間「記憶體設定檔」下，建立稱為「使用中的記憶體位元組」的自訂計量。 計量有一個稱為「處理序」的維度。 針對指定的時間戳記，我們會為兩個不同的處理序發出計量值：

```json
{
    "time": "2018-08-20T11:25:20-7:00",
    "data": {

      "baseData": {

        "metric": "Memory Bytes in Use",
        "namespace": "Memory Profile",
        "dimNames": [
          "Process"        ],
        "series": [
          {
            "dimValues": [
              "ContosoApp.exe"
            ],
            "min": 10,
            "max": 89,
            "sum": 190,
            "count": 4
          },
          {
            "dimValues": [
              "SalesApp.exe"
            ],
            "min": 10,
            "max": 23,
            "sum": 86,
            "count": 4
          }
        ]
      }
    }
  }
```
> [!NOTE]
> Application Insights 這個 Windows Azure 診斷擴充功能與 InfluxData Telegraf 代理程式均已設定，可對正確的區域端點發出計量值，並執行每次發出中的上述所有屬性。
>
>

## <a name="custom-metric-definitions"></a>自訂計量定義
在將自訂計量發出之前，並不需要在 Azure 監視器中預先定義。 由於發佈的每個計量資料點都包含命名空間、名稱與維度資訊，首次將自訂計量發出至 Azure 監視器時，就會自動建立計量定義。 您隨後可以在會透過計量定義發出計量的任何資源上，探索此計量定義。

> [!NOTE]
> Azure 監視器尚不支援定義自訂計量的「單位」。

## <a name="using-custom-metrics"></a>使用自訂計量
將自訂計量提交至 Azure 監視器之後，您就可以透過 Azure 入口網站瀏覽，透過 Azure 監視器 REST API 查詢，或對其建立警示，讓您在符合特定條件時收到通知。
### <a name="browse-your-custom-metrics-via-the-azure-portal"></a>透過 Azure 入口網站瀏覽自訂計量
1.  移至 [Azure 入口網站](https://portal.azure.com)
2.  選取 [監視] 刀鋒視窗
3.  按一下 [計量]
4.  選取您已對其發出自訂計量的資源
5.  選取自訂計量的計量命名空間
6.  選取自訂計量

## <a name="supported-regions"></a>支援的區域
在公開預覽期間，發佈自訂計量的功能只能在 Azure 區域的子集中使用。 這表示只能針對其中一個支援區域的資源發佈計量。 下表列出自訂計量的一組支援 Azure 區域，以及應針對這些區域中的資源發佈的對應端點計量。

|Azure 區域|區域端點前置詞|
|---|---|
|美國東部|https://eastus.monitoring.azure.com/|
|美國中南部|https://southcentralus.monitoring.azure.com/|
|美國中西部|https://westcentralus.monitoring.azure.com/|
|美國西部 2|https://westus2.monitoring.azure.com/|
|東南亞|https://southeastasia.monitoring.azure.com/|
|北歐|https://northeurope.monitoring.azure.com/|
|西歐|https://westeurope.monitoring.azure.com/|

## <a name="quotas-and-limits"></a>配額和限制
Azure 監視器會對自訂計量加諸下列使用量限制。

|類別|限制|
|---|---|
|使用中的時間序列/訂用帳戶/區域|50,000|
|每個計量的維度索引鍵|10|
|計量命名空間、計量名稱、維度索引鍵與維度值的字串長度|256 個字元|
使用中的時間序列定義為計量、維度索引鍵、維度值 (含有過去 12 小時內所發佈計量值) 的任意獨特組合。

## <a name="next-steps"></a>後續步驟
使用來自不同服務的自訂計量 
 - [虛擬機器](metrics-store-custom-guestos-resource-manager-vm.md)
 - [虛擬機器擴展集](metrics-store-custom-guestos-resource-manager-vmss.md)
 - [虛擬機器 (傳統)](metrics-store-custom-guestos-classic-vm.md)
 - [使用 Telegraf 代理程式的 Linux 虛擬機器](metrics-store-custom-linux-telegraf.md)
 - [REST API](metrics-store-custom-rest-api.md)
 - [雲端服務 (傳統)](metrics-store-custom-guestos-classic-cloud-service.md)
 