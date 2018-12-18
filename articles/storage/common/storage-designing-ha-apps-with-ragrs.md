---
title: 使用 Azure 讀取權限異地備援儲存體 (RA-GRS) 設計高可用性應用程式 | Microsoft Docs
description: 如何使用 Azure RA-GRS 儲存體來設計高可用性應用程式的架構，使其有足夠的彈性來處理中斷。
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 03/21/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: afcda23faf4e9f0999442fa91d3c016e446c04db
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39524537"
---
# <a name="designing-highly-available-applications-using-ra-grs"></a>使用 RA-GRS 設計高可用性應用程式

雲端式基礎結構 (例如 Azure 儲存體) 的常見功能是提供高可用性平台來裝載應用程式。 雲端式應用程式的開發人員必須仔細考慮如何運用此平台，來為使用者提供高可用性應用程式。 本文著重於開發人員如何利用讀取權限異地備援儲存體 (RA-GRS)，來確定其 Azure 儲存體應用程式的高可用性。

[!INCLUDE [storage-common-redundancy-options](../../../includes/storage-common-redundancy-options.md)]

本文的重點在於 GRS 和 RA-GRS。 使用 GRS 時，會在您設定儲存體帳戶時選取的主要區域中保留三份您的資料。 另外會以非同步方式在 Azure 所指定的次要區域中保留三個額外複本。 RA-GRS 提供具有次要複本讀取存取權的異地備援儲存體。

如需主要區域與次要區域配對的相關資訊，請參閱[商務持續性和災害復原 (BCDR)：Azure 配對的區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。

本文提供一些程式碼片段，並在結尾處提供完整範例的連結，可供您下載並執行。

> [!NOTE]
> 「Azure 儲存體」現已支援使用區域備援儲存體 (ZRS) 來建置高可用性應用程式。 ZRS 針對許多應用程式的備援需求，提供一個簡單的解決方案。 ZRS 可以針對硬體故障或影響單一資料中心的災難性災害提供防護。 如需詳細資訊，請參閱[區域備援儲存體 (ZRS)：高可用性 Azure 儲存體應用程式](storage-redundancy-zrs.md)。

## <a name="key-features-of-ra-grs"></a>RA-GRS 的主要功能

針對 RA-GRS 設計您的應用程式時請記住這些要點：

* Azure 儲存體會維護您在主要區域、次要區域中儲存之資料的唯讀副本。 如先前所述，儲存體服務會決定次要區域的位置。

* 唯讀複本與主要區域中的資料是[最終一致](https://en.wikipedia.org/wiki/Eventual_consistency)的。

* 針對 Blob、資料表和佇列，您可以查詢次要區域來取得 [上次同步處理時間] 值，讓您知道上次從主要區域複寫到次要區域是在何時發生 (Azure 檔案服務不支援此動作，它目前沒有 RA-GRS 備援)。

* 您可以使用儲存體用戶端程式庫，來與主要或次要區域中的資料進行互動。 如果對主要區域的讀取要求逾時，您也可以將讀取要求自動重新導向到次要區域。

* 如果產生會影響主要區域中資料可存取性的主要問題，Azure 團隊可能會觸發異地複寫容錯移轉，此時指向主要區域的 DNS 項目將會變更為指向次要區域。

* 發生異地複寫容錯移轉時，Azure 將會選取新的次要位置並將資料複寫到該位置，然後將次要 DNS 項目指向它。 次要端點要到儲存體帳戶複寫完成之後才能使用。 如需詳細資訊，請參閱[如果 Azure 儲存體發生中斷怎麼辦](https://docs.microsoft.com/azure/storage/storage-disaster-recovery-guidance)。

## <a name="application-design-considerations-when-using-ra-grs"></a>使用 RA-GRS 時的應用程式設計考量

本文的目的是示範如何設計即使主要資料中心發生重大災害，仍能繼續運作 (儘管容量有限) 的應用程式。 您可以設計您的應用程式，藉由當從主要區域讀取發生干擾問題時從次要區域讀取，來處理暫時性或長時間執行的問題。 當主要區域再次可用時，您的應用程式可以回到從主要區域讀取。

### <a name="using-eventually-consistent-data"></a>使用最終一致的資料

這個建議的解決方案假設可以接受將可能過時的資料傳回呼叫的應用程式。 由於次要區域中的資料最終會一致，因此在次要區域的更新完成複寫之前，主要區域可能會無法存取。

例如，假設您的客戶成功提交更新，但是在更新傳播到次要區域之前主要區域會失敗。 當客戶要求讀回資料，他會從次要區域接收到過時的資料，而不是更新的資料。 設計您的應用程式時，您必須決定是否可接受此情況，如果可以，您該如何傳達給客戶。 

在本文後續內容中，我們會看到如何檢查次要資料的上次同步處理時間，以查看該次要資料是否為最新狀態。

### <a name="handling-services-separately-or-all-together"></a>個別處理服務或一併處理所有服務

儘管不太可能，但某一個服務可能會在其他服務仍可完全運作時變成無法使用。 您可以針對每個服務 (Blob、佇列、資料表) 個別處理重試和唯讀模式，或者以一般方式一併處理所有儲存體服務的重試。

例如，如果您在應用程式中使用佇列和 Blob，您就能決定放入不同程式碼來處理這其中每一個項目的可重試錯誤。 接著，如果您從 Blob 服務進行重試，但佇列服務仍在運作，則只有處理 Blob 的應用程式組件會受到影響。 如果您決定要以一般方式處理所有儲存體服務重試，而且對 Blob 服務的呼叫會傳回可重試的錯誤，則對 Blob 服務和佇列服務的要求都將受到影響。

這最終取決於應用程式的複雜性。 在您偵測到主要區域中發生任何儲存體服務的問題時，您可能會決定不要處理服務所造成的失敗，而是改為將所有儲存體服務的讀取要求重新導向到次要區域，並在唯讀模式中執行應用程式。

### <a name="other-considerations"></a>其他考量

以下是我們將在本文其餘內容中討論的其他考量。

*   使用斷路器模式處理讀取要求的重試

*   最終一致的資料和上次同步處理時間

*   測試

## <a name="running-your-application-in-read-only-mode"></a>在唯讀模式中執行您的應用程式

若要使用 RA-GRS 儲存體，您必須能夠同時處理失敗的讀取要求和失敗的更新要求 (透過更新，在此案例中表示插入、更新和刪除)。 如果主要資料中心失敗，可將讀取要求重新導向到次要資料中心。 不過，因為次要資料中心是唯讀的，所以無法將更新要求重新導向到次要資料中心。 基於這個理由，您必須設計您的應用程式在唯讀模式中執行。

例如，您可以設定旗標，在將任何更新要求提交到 Azure 儲存體之前先加以檢查。 當其中一個更新要求傳入時，您可以略過它，並將適當的回應傳回給客戶。 您甚至可以在問題解決之前完全停用某些功能，並通知使用者，這些功能暫時無法使用。

如果您決定要個別處理每個服務的錯誤，也需要處理服務在唯讀模式中執行您應用程式的能力。 例如，您可以對可啟用和停用的每個服務加上唯讀旗標。 然後您可以在程式碼中的適當位置處理旗標。

能夠在唯讀模式中執行您的應用程式有另一個好處 - 這讓您能夠在主要應用程式升級期間確保有限的功能。 您可以觸發應用程式在唯讀模式中執行並指向次要資料中心，確保當您進行升級時，不會有人正在存取主要區域中的資料。

## <a name="handling-updates-when-running-in-read-only-mode"></a>在唯讀模式中執行時處理更新

有許多方法可在唯讀模式中執行時處理更新要求。 我們不會全面討論此問題，但一般來說，有一些模式是您需要考量的。

1.  您可以回應您的使用者，並告訴他們，您目前不會接收更新。 例如，連絡人管理系統能夠讓客戶存取連絡人資訊，但無法進行更新。

2.  您可以將更新加入其他區域中的佇列。 在此情況下，您會將擱置中的更新要求寫入不同區域中的佇列，然後想辦法在主要資料中心再次上線之後處理這些要求。 在此案例中，您應該讓客戶知道已將要求的更新排入佇列，以供稍後處理。

3.  您可以將更新寫入其他區域中的儲存體帳戶。 然後在主要資料中心再度上線時，您就能根據資料結構，想辦法將這些更新合併到主要資料。 例如，如果您建立了名稱中含有日期/時間戳記的不同檔案，就能將這些檔案複製回主要區域。 這適用於某些工作負載，例如記錄和 iOT 資料。

## <a name="handling-retries"></a>處理重試

您如何知道哪些錯誤是可重試的？ 這取決於儲存體用戶端程式庫。 例如，404 錯誤 (找不到資源) 是不可重試的，因為重試很可能不會順利完成。 相反地，500 錯誤是可重試的，因為它是伺服器錯誤，而且可能只是暫時性問題。 如需詳細資訊，請參閱 .NET 儲存體用戶端程式庫中的 [ExponentialRetry 類別的開放原始程式碼 (英文)](https://github.com/Azure/azure-storage-net/blob/87b84b3d5ee884c7adc10e494e2c7060956515d0/Lib/Common/RetryPolicies/ExponentialRetry.cs) (尋找 ShouldRetry 方法)。

### <a name="read-requests"></a>讀取要求

如果主要儲存體發生問題，就可以將讀取要求重新導向到次要儲存體。 如前面[最終使用一致的資料](#using-eventually-consistent-data)中所述，您的應用程式必須可以接受可能讀取到過時的資料。 如果您使用儲存體用戶端程式庫來存取 RA-GRS 資料，就可以藉由將 **LocationMode** 屬性值設為下列其中一個值來指定讀取要求的重試行為：

*   **PrimaryOnly** (預設值)

*   **PrimaryThenSecondary**

*   **SecondaryOnly**

*   **SecondaryThenPrimary**

當您將 **LocationMode** 設為 **PrimaryThenSecondary** 時，如果對主要端點的最初讀取要求失敗且產生可重試的錯誤，則用戶端會自動對次要端點提出另一個讀取要求。 如果錯誤是伺服器逾時，則用戶端在接收到服務所提供的可重試錯誤之前，必須等候逾時到期。

當您決定如何回應可重試的錯誤時，基本上有兩種情況需要考量：

*   這是一個獨立的問題，而對主要端點的後續要求將不會傳回可重試的錯誤。 舉例來說，這可能發生在有暫時性網路錯誤時。

    在此案例中，將 **LocationMode** 設為 **PrimaryThenSecondary** 並不會對效能產生顯著的負面影響，因為這很少發生。

*   這是一個與主要區域中至少一個儲存體服務相關的問題，而對主要區域中該服務的所有後續要求很可能會在一段時間內傳回可重試的錯誤。 如果主要區域完全無法存取，即為一例。

    在此案例中，會對效能產生負面影響，因為您的所有讀取要求都會先嘗試主要端點、等候逾時到期，然後切換到次要端點。

針對這些情況，您應該確認主要端點會持續發生問題，並藉由將 **LocationMode** 屬性設為 **SecondaryOnly**，直接將所有讀取要求傳送到次要端點。 此時，您也應該將應用程式變更為在唯讀模式中執行。 這種方法稱為[斷路器模式 (英文)](https://msdn.microsoft.com/library/dn589784.aspx)。

### <a name="update-requests"></a>更新要求

斷路器模式也適用於更新要求。 不過，無法將更新要求重新導向到唯讀的次要儲存體。 針對這些要求，您應該讓 **LocationMode**屬性維持 **PrimaryOnly** (預設值) 的設定。 若要處理這些錯誤，您可以將計量套用到這些要求 (例如，一列 10 個失敗)，並在到達臨界值時，將應用程式切換回唯讀模式。 您可以使用與下一個關於斷路器模式的小節中所述的相同方法來返回更新模式。

## <a name="circuit-breaker-pattern"></a>斷路器模式

在您的應用程式中使用斷路器模式，可防止重複重試很可能會失敗的作業。 它可以讓應用程式繼續執行，而不會在以指數方式重試作業時佔用時間。 它也會偵測到已修正錯誤的時間，應用程式可於該時間點再次重試此作業。

### <a name="how-to-implement-the-circuit-breaker-pattern"></a>如何實作斷路器模式

若要確認主要端點持續發生問題，您可以監視用戶端發生可重試錯誤的頻率。 由於每個案例不同，因此，您必須決定要用於決定切換到次要端點，並在唯讀模式中執行應用程式的臨界值。 例如，如果沒有成功的資料列中有 10 個錯誤，則您可能決定執行切換。 另一個範例是假設 2 分鐘內有 90 % 的要求失敗，就進行切換。

針對第一個案例，您只需保留失敗的計數，而且如果在到達最大值之前成功，則會將計數設定為零。 針對第二個案例，有一個實作它的方式是使用 MemoryCache 物件 (在 .NET 中)。 針對每個要求，將 CacheItem 新增到快取、將值設為成功 (1) 或失敗 (0)，然後將到期時間設定為從目前開始 2 分鐘 (或任何時間限制)。 到達項目的到期時間時，即會自動移除項目。 這將會為您提供一個 2 分鐘循環的視窗。 每當您對儲存體服務提出要求時，您要先跨 MemoryCache 物件使用 Linq 查詢，藉由加總這些值並除以計數來計算成功百分比。 當成功百分比低於某個臨界值 (例如 10%) 時，將讀取要求的 **LocationMode** 屬性設為 **SecondaryOnly**，並將應用程式切換到唯讀模式，然後再繼續。

用來判斷何時要進行切換的錯誤臨界值可能會因應用程式中的服務不同而有所差異，因此您應該考慮讓它們成為可設定的參數。 這也是您決定個別處理每個服務之可重試錯誤的地方 (或當成一個來處理)，如先前所述。

另一個考量是如何處理應用程式的多個執行個體，以及當您在每個執行個體中偵測到可重試的錯誤時該怎麼辦。 例如，您可能有 20 個 VM 正載入相同的應用程式來執行。 您要個別處理每個執行個體嗎？ 如果有一個執行個體開始發生問題，您想要將回應只限制在該執行個體，或者您要在一個執行個體發生問題時，嘗試以相同方式回應所有執行個體？ 個別處理執行個體會比嘗試在它們之間協調回應更簡單，但該如何做則取決於您的應用程式架構。

### <a name="options-for-monitoring-the-error-frequency"></a>監視錯誤頻率的選項

您有三個主要選項可用來監視主要區域中的重試頻率，以判斷何時要切換到次要區域，並將應用程式變更為在唯讀模式中執行。

*   針對您傳遞到儲存體要求的 [**OperationContext**](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.aspx) 物件上的 [**Retrying**](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx) 事件新增處理常式 - 這是本文所示的方法，並會在隨附的範例中使用。 每當用戶端重試要求時，這些事件即會觸發，讓您能夠追蹤用戶端在主要端點上發生可重試錯誤的頻率。

    ```csharp 
    operationContext.Retrying += (sender, arguments) =>
    {
        // Retrying in the primary region
        if (arguments.Request.Host == primaryhostname)
            ...
    };
    ```

*   在自訂重試原則的 [**Evaluate**](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx) 方法中，您可以在每次進行重試時執行自訂程式碼。 除了記錄重試發生的時間，這也能讓您有機會修改重試行為。

    ```csharp 
    public RetryInfo Evaluate(RetryContext retryContext,
    OperationContext operationContext)
    {
        var statusCode = retryContext.LastRequestResult.HttpStatusCode;
        if (retryContext.CurrentRetryCount >= this.maximumAttempts
            || ((statusCode >= 300 && statusCode < 500 && statusCode != 408)
            || statusCode == 501 // Not Implemented
            || statusCode == 505 // Version Not Supported
            ))
        {
            // Do not retry
            return null;
        }

        // Monitor retries in the primary location
        ...

        // Determine RetryInterval and TargetLocation
        RetryInfo info =
            CreateRetryInfo(retryContext.CurrentRetryCount);

        return info;
    }
    ```

*   第三個方法是在持續使用虛設的讀取要求 (例如讀取小型 blob) 來 ping 您主要儲存體端點的應用程式中，實作自訂的監視元件來判斷其健康狀態。 這會佔用某些資源，但時間不是很長。 在探索到達到您臨界值的問題時，您接著可以執行切換到 **SecondaryOnly** 和唯讀模式。

您會想要在某個時間點切換回來，以使用主要端點並允許更新。 如果使用上述前兩個方法的其中一個，您可能只需切換回主要端點，並在執行任意選取的時間量或作業數目之後啟用更新模式。 您接著可以讓它重新進行重試邏輯。 如果已修正問題，它將會繼續使用主要端點，並允許更新。 如果仍有問題，它會在無法達到您所設定的準則時，再次切換回次要端點和唯讀模式。

針對第三個案例，當 ping 主要儲存體端點再度變為成功時，您就能觸發切換回 **PrimaryOnly**，並繼續允許更新。

## <a name="handling-eventually-consistent-data"></a>處理最終一致的資料

RA-GRS 的運作方式是將交易從主要區域複寫到次要區域。 此複寫程序可保證次要區域中的資料是*最終一致*的。 這表示，主要區域中的所有交易最終都會出現在次要區域中，但有可能會延遲出現，而且不保證交易會以原本在主要區域中套用它們的相同順序到達次要區域。 如果您的交易不按順序到達次要區域，則您可能要考慮讓次要區域中的資料處於不一致狀態，直到服務更新為止。

下表所示的範例是，當您更新員工的詳細資料，使她成為「系統管理員」角色的成員時，可能會發生什麼情況。 基於此範例，這會要求您更新**員工**實體，並利用系統管理員總數的計數來更新**系統管理員角色**實體。 請注意，如何在次要區域中不按順序套用更新。

| <bpt id="p1">**</bpt>Time<ept id="p1">**</ept> | **交易**                                            | **複寫**                       | **上次同步處理時間** | **結果** |
|----------|------------------------------------------------------------|---------------------------------------|--------------------|------------| 
| T0       | 交易 A： <br> 會在主要區域中 <br> 插入員工實體 |                                   |                    | 交易 A 已插入至主要區域，<br> 但尚未複寫。 |
| T1       |                                                            | 交易 A <br> 已複寫到<br> 次要區域 | T1 | 交易 A 已複寫到次要區域。 <br>已更新上次同步處理時間。    |
| T2       | 交易 B：<br>更新<br> 主要區域中的<br> 員工實體  |                                | T1                 | 交易 B 已寫入主要區域，<br> 但尚未複寫。  |
| T3       | 交易 C：<br> 更新 <br>administrator<br>角色實體，位於<br>primary |                    | T1                 | 交易 C 已寫入主要區域，<br> 但尚未複寫。  |
| *T4*     |                                                       | 交易 C <br>已複寫到<br> 次要區域 | T1         | 交易 C 已複寫到次要區域。<br>無法更新 LastSyncTime，因為 <br>尚未複寫交易 B。|
| *T5*     | 從次要區域 <br>讀取實體                           |                                  | T1                 | 您取得員工實體的過時值， <br> 因為交易 B 尚未 <br> 複寫。 您取得系統管理員角色實體<br> 的新值，因為 C<br> 已複寫。 上次同步處理時間仍然尚未<br> 更新，因為交易 B<br> 尚未複寫。 您可以說<br>系統管理員角色實體不一致， <br>因為實體日期/時間是在 <br>上次同步處理時間之後。 |
| *T6*     |                                                      | 交易 B<br> 已複寫到<br> 次要區域 | T6                 | *T6* - 透過 C 的所有交易都 <br>已複寫，上次同步處理時間<br> 已更新。 |

在此範例中，假設用戶端會在 T5 從次要區域切換到讀取。 它可以在此時順利讀取**系統管理員角色**實體，但實體包含的系統管理員計數值與**員工**實體數目不一致，後者在此時標示為次要區域中的系統管理員。 您的用戶端可能只會顯示此值，存有資訊不一致的風險。 或者，用戶端可能嘗試判斷**系統管理員角色**處於可能不一致的狀態，因為更新並未按順序進行，然後通知使用者此一事實。

若要辨識其中可能含有不一致的資料，用戶端可以使用「上次同步處理時間」的值，你可以隨時查詢儲存體服務來取得此值。 這會告訴您次要區域中的資料上次保持一致的時間，以及在此時間點之前服務套用所有交易的時間。 在上述範例中，當服務在次要區域中插入**員工**實體之後，就會將上次同步處理時間設為 *T1*。 它會保持在 *T1*，直到將其設為 *T6* 之後，服務更新次要區域中的**員工**實體為止。 如果用戶端會在其讀取 *T5* 上的實體時擷取上次同步處理時間，就能與實體上的時間戳記進行比較。 如果實體上的時間戳記晚於上次同步處理時間，則該實體可能會處於不一致狀態，而您就能針對應用程式採取適當的動作。 您必須知道上次對主要區域完成更新的時間，才能使用此欄位。

## <a name="testing"></a>測試

請務必測試應用程式在發生可重試的錯誤時會如預期般運作。 例如，您需要測試應用程式會在偵測到問題時切換到次要區域並進入唯讀模式，然後在主要區域再度變成可供使用時切換回來。 若要這樣做，您需要想辦法模擬可重試的錯誤，並控制其發生頻率。

您可以使用 [Fiddler (英文)](http://www.telerik.com/fiddler)，來攔截與修改指令碼中的 HTTP 回應。 此指令碼可以識別來自您主要端點的回應，並將 HTTP 狀態碼變更為儲存體用戶端程式庫可辨識為可重試錯誤的狀態碼。 此程式碼片段示範一個簡單的 Fiddler 指令碼範例，來攔截對 **employeedata** 資料表之讀取要求的回應，以傳回 502 狀態：

```java
static function OnBeforeResponse(oSession: Session) {
    ...
    if ((oSession.hostname == "\[yourstorageaccount\].table.core.windows.net")
      && (oSession.PathAndQuery.StartsWith("/employeedata?$filter"))) {
        oSession.responseCode = 502;
    }
}
```

您可以擴充此範例，以攔截範圍較大的要求，而且只需變更其中的 **responseCode**，就能進一步模擬真實世界的案例。 如需自訂 Fiddler 指令碼的詳細資訊，請參閱 Fiddler 文件中的[修改要求或回應 (英文)](http://docs.telerik.com/fiddler/KnowledgeBase/FiddlerScript/ModifyRequestOrResponse)。

如果您已讓可用來將應用程式切換到唯讀模式的臨界值變成可設定的，就能更輕易地使用非實際執行的交易量來測試此行為。

## <a name="next-steps"></a>後續步驟

* 如需讀取存取異地備援的詳細資訊 (包括另一個如何設定 LastSyncTime 的範例)，請參閱 [Microsoft Azure 儲存體備援選項與讀取權限異地備援儲存體](https://blogs.msdn.microsoft.com/windowsazurestorage/2013/12/11/windows-azure-storage-redundancy-options-and-read-access-geo-redundant-storage/)。

* 如需示範如何在主要和次要端點之間來回切換的完整範例，請參閱 [Azure 範例 - 搭配 RA-GRS 儲存體使用斷路器模式 (英文)](https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs)。
