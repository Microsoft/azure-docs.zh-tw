---
title: 使用可靠的集合 | Microsoft Docs
description: 了解使用可靠的集合的最佳做法。
services: service-fabric
documentationcenter: .net
author: rajak
manager: timlt
editor: ''
ms.assetid: 39e0cd6b-32c4-4b97-bbcf-33dad93dcad1
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/19/2017
ms.author: rajak
ms.openlocfilehash: 2568e116fdb3f80976d49787877d2ecf68f128ef
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34210812"
---
# <a name="working-with-reliable-collections"></a>使用可靠的集合
Service Fabric 透過可靠的集合向 .NET 開發人員提供具狀態的程式設計模型。 具體來說，Service Fabric 提供了可靠的字典和可靠的佇列類別。 當您使用這些類別時，您的狀態是分割的 (延展性)、複寫的 (可用性)，且在分割區內交易 (ACID 語意)。 讓我們看看可靠字典物件的一般用法，並看看它究竟做些什麼。

```csharp

///retry:

try {
   // Create a new Transaction object for this partition
   using (ITransaction tx = base.StateManager.CreateTransaction()) {
      // AddAsync takes key's write lock; if >4 secs, TimeoutException
      // Key & value put in temp dictionary (read your own writes),
      // serialized, redo/undo record is logged & sent to
      // secondary replicas
      await m_dic.AddAsync(tx, key, value, cancellationToken);

      // CommitAsync sends Commit record to log & secondary replicas
      // After quorum responds, all locks released
      await tx.CommitAsync();
   }
   // If CommitAsync not called, Dispose sends Abort
   // record to log & all locks released
}
catch (TimeoutException) {
   await Task.Delay(100, cancellationToken); goto retry;
}
```

可靠的字典物件上的所有作業 (除了無法復原的 ClearAsync)，都需要一個 ITransaction 物件。 這個物件和您在單一分割區內對任何可靠的字典和/或可靠的佇列物件嘗試進行的任何及所有變更具有關聯性。 您是透過呼叫分割區的 StateManager 的 CreateTransaction 方法，取得 ITransaction 物件。

在上面的程式碼中，ITransaction 物件會傳遞至可靠的字典的 AddAsync 方法。 就內部而言，接受索引鍵的字典方法會採用與索引鍵相關聯的讀取器/寫入器鎖定。 如果此方法修改索引鍵的值，這個方法就會在索引鍵上使用寫入鎖定；如果方法只讀取索引鍵的值，就會在索引鍵上使用讀取鎖定。 因為 AddAsync 會將索引鍵值修改成新的傳入值，所以使用索引鍵的寫入鎖定。 所以，如果有 2 (或多個) 執行緒嘗試在同一時間加入相同的索引鍵值，一個執行緒就會取得寫入鎖定，而另一個執行緒就會封鎖。 方法預設最多封鎖 4 秒以取得鎖定，4 秒後方法就會擲回 TimeoutException。 如果喜歡的話，方法多載的存在可讓您傳遞明確的逾時值。

通常，您撰寫程式碼回應 TimeoutException 的方式是攔截它，然後重試整個作業 (如上面的程式碼所示)。 在我的簡單程式碼中，只呼叫 Task.Delay 每次傳遞 100 毫秒。 但在實際狀況裡，最好還是改用某種指數型的撤退延遲。

一旦取得鎖定，AddAsync 就會在與 ITransaction 物件相關聯的內部暫存字典中加入索引鍵和值物件參考。 這就完成了讀取自己撰寫的語意。 也就是說，在您呼叫 AddAsync 之後，稍後對 TryGetValueAsync 的呼叫 (使用相同的 ITransaction 物件) 會傳回值，即使您尚未認可交易。 接下來，AddAsync 會將索引鍵和值物件序列化為位元組陣列，並將這些位元組陣列附加至本機節點的記錄檔。 最後，AddAsync 會將位元組陣列傳送給所有次要複本，讓它們有相同的索引鍵/值資訊。 即使索引鍵/值資訊已寫入記錄檔，在相關交易獲認可之前，資訊都不會被視為字典的一部分。

在上述的程式碼中，呼叫 CommitAsync 即認可所有的交易作業。 具體來說，它會將認可資訊附加到本機節點的記錄檔，也會將認可記錄傳送給所有的次要複本。 一旦回覆了複本的仲裁 (多數)，則所有資料變更就會視為永久性，並且會釋出透過 ITransaction 物件所操作的任何索引鍵相關聯鎖定，讓其他執行緒/交易可以操作相同的索引鍵及其值。

如未呼叫 CommitAsync (通常是因為擲回例外狀況)，則會處置 ITransaction 物件。 在處置未認可的 ITransaction 物件時，Service Fabric 會中止將資訊附加到本機節點的記錄檔，且不需要傳送任何資訊至任何次要複本。 然後會釋出透過交易操作的任何與索引鍵相關聯的鎖定。

## <a name="common-pitfalls-and-how-to-avoid-them"></a>常見陷阱以及如何避免
現在您已了解可靠的集合在內部的運作方式，讓我們看看其中一些常見的誤用。 參閱下列程式碼：

```csharp
using (ITransaction tx = StateManager.CreateTransaction()) {
   // AddAsync serializes the name/user, logs the bytes,
   // & sends the bytes to the secondary replicas.
   await m_dic.AddAsync(tx, name, user);

   // The line below updates the property’s value in memory only; the
   // new value is NOT serialized, logged, & sent to secondary replicas.
   user.LastLogin = DateTime.UtcNow;  // Corruption!

   await tx.CommitAsync();
}
```

使用一般的 .NET 字典時，您可以在字典中加入索引鍵/值，然後變更屬性的值 (例如 LastLogin)。 不過，這段程式碼和可靠的字典不會合作無間。 我們稍早討論過：呼叫 AddAsync 會將索引鍵/值物件序列化成位元組陣列，然後將陣列儲存到本機檔案，並將它們傳送到次要複本。 稍後如果變更屬性，只會變更記憶體中的屬性值，不會影響本機檔案或傳送到複本的資料。 如果處理序損毀，記憶體中內容將全部遺失。 啟動新的處理序，或另一個複本變成主要複本時，使用的就是舊的屬性值。

我一再強調上面這種錯誤是很容易發生的。 而且您只有在處理序當機時才會發現錯誤。 撰寫程式碼的正確方式其實只要反轉兩行即可︰


```csharp

using (ITransaction tx = StateManager.CreateTransaction()) {
   user.LastLogin = DateTime.UtcNow;  // Do this BEFORE calling AddAsync
   await m_dic.AddAsync(tx, name, user);
   await tx.CommitAsync();
}
```

這是另一個常見的錯誤︰

```csharp

using (ITransaction tx = StateManager.CreateTransaction()) {
   // Use the user’s name to look up their data
   ConditionalValue<User> user =
      await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (user.HasValue) {
      // The line below updates the property’s value in memory only; the
      // new value is NOT serialized, logged, & sent to secondary replicas.
      user.Value.LastLogin = DateTime.UtcNow; // Corruption!
      await tx.CommitAsync();
   }
}
```

同樣地，使用一般的 .NET 字典時，上面的程式碼以常見的模式正常運作：開發人員使用索引鍵查詢值。 如果值存在，開發人員就變更屬性的值。 不過，使用可靠的集合，這段程式碼會出現前面討論過的同樣問題：**物件一旦給了可靠的集合，就「無法」修改。**

在可靠的集合中更新值的正確方式，是取得現有值的參考，並考慮這個參考所參考的物件為不可變。 接著，建立新的物件，這是原始物件的完全相同複本。 現在，您可以修改這個新物件的狀態，將新物件寫入集合，讓它序列化為位元組陣列，附加至本機檔案並傳送到複本。 認可變更之後，記憶體中的物件、本機檔案及所有複本都會有完全一致的狀態。 大功告成！

下列程式碼會示範在可靠的集合中更新值的正確方式︰

```csharp

using (ITransaction tx = StateManager.CreateTransaction()) {
   // Use the user’s name to look up their data
   ConditionalValue<User> currentUser =
      await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (currentUser.HasValue) {
      // Create new user object with the same state as the current user object.
      // NOTE: This must be a deep copy; not a shallow copy. Specifically, only
      // immutable state can be shared by currentUser & updatedUser object graphs.
      User updatedUser = new User(currentUser);

      // In the new object, modify any properties you desire
      updatedUser.LastLogin = DateTime.UtcNow;

      // Update the key’s value to the updateUser info
      await m_dic.SetValue(tx, name, updatedUser);

      await tx.CommitAsync();
   }
}
```

## <a name="define-immutable-data-types-to-prevent-programmer-error"></a>定義不可變的資料類型，以防止程式設計人員犯錯。
理想情況下，我們希望編譯器能在您不小心產生改變物件狀態的程式碼，而這個物件又不該改變時報告錯誤。 但是 C# 編譯器做不到這一點。 所以，為避免潛在的程式設計人員錯誤，我們強烈建議您將可靠集合所使用的類型定義為不可變的類型。 具體來說，這表示您要堅持核心值類型 (例如數字 [Int32、UInt64 等等]、DateTime、Guid、TimeSpan 等等)。 當然，您也可以使用字串。 最好是避免集合屬性，因為將其序列化和還原序列化經常會降低效能。 不過，如果您想要使用集合屬性，強烈建議您使用 .NET 的不可變集合程式庫 ([System.Collections.Immutable](https://www.nuget.org/packages/System.Collections.Immutable/))。 您可以從 http://nuget.org 下載這個程式庫。另外，也建議您盡可能密封類別，並將欄位變成唯讀。

以下的 UserInfo 類型會示範如何利用上述建議定義不可變的類型。

```csharp

[DataContract]
// If you don’t seal, you must ensure that any derived classes are also immutable
public sealed class UserInfo {
   private static readonly IEnumerable<ItemId> NoBids = ImmutableList<ItemId>.Empty;

   public UserInfo(String email, IEnumerable<ItemId> itemsBidding = null) {
      Email = email;
      ItemsBidding = (itemsBidding == null) ? NoBids : itemsBidding.ToImmutableList();
   }

   [OnDeserialized]
   private void OnDeserialized(StreamingContext context) {
      // Convert the deserialized collection to an immutable collection
      ItemsBidding = ItemsBidding.ToImmutableList();
   }

   [DataMember]
   public readonly String Email;

   // Ideally, this would be a readonly field but it can't be because OnDeserialized
   // has to set it. So instead, the getter is public and the setter is private.
   [DataMember]
   public IEnumerable<ItemId> ItemsBidding { get; private set; }

   // Since each UserInfo object is immutable, we add a new ItemId to the ItemsBidding
   // collection by creating a new immutable UserInfo object with the added ItemId.
   public UserInfo AddItemBidding(ItemId itemId) {
      return new UserInfo(Email, ((ImmutableList<ItemId>)ItemsBidding).Add(itemId));
   }
}
```

ItemId 類型也是不可變的類型，如下所示︰

```csharp

[DataContract]
public struct ItemId {

   [DataMember] public readonly String Seller;
   [DataMember] public readonly String ItemName;
   public ItemId(String seller, String itemName) {
      Seller = seller;
      ItemName = itemName;
   }
}
```

## <a name="schema-versioning-upgrades"></a>結構描述版本控制 (升級)
就內部而言，可靠的集合會使用 .NET 的 DataContractSerializer 序列化物件。 序列化的物件會保存在主要複本的本機磁碟中，並傳送至次要複本。 隨著您的服務日趨成熟，您可能會想要變更服務需要的資料種類 (結構描述)。 您必須十二萬分地謹慎對待資料的版本控制方法。 首先也是最重要的，您必須永遠有能力還原序列化舊的資料。 具體來說，這表示您的還原序列化程式碼必須具有無限回溯相容性︰服務程式碼的版本 333 必須能夠操作 5 年前放在可靠的集合中，第 1 版的服務程式碼資料。

而且，服務程式碼一次只能升級一個網域。 所以，在升級期間，您會同時執行兩個不同版本的服務程式碼。 您必須避免新版本的服務程式碼使用新的結構描述，因為舊版的服務程式碼可能無法處理新的結構描述。 您應該盡可能將每個版本的服務都設計成向前相容 1 個版本。 具體來說，這表示 V1 的服務程式碼只要能夠略過它不會明確處理的任何結構描述元素即可。 不過，它必須能夠儲存它不明確了解的任何資料，在更新字典索引鍵或值時只需將它寫回即可。

> [!WARNING]
> 雖然您可以修改索引鍵的結構描述，但您必須確保索引鍵雜湊程式碼和 equals 演算法是穩定的。 如果您變更這些演算法其中一個的運作方式，您就再也無法在可靠的字典內查詢索引鍵。
>
>

或者，您也可以執行通稱為 2 階段升級的功能。 使用 2 階段升級，服務可從 V1 升級成 V2：V2 包含知道如何處理新結構描述變更的程式碼，但這段程式碼不會執行。 當 V2 程式碼讀取 V1 資料時，它會在其上操作並寫入 V1 資料。 然後，在跨所有升級網域的升級都完成之後，您就可以通知執行中的 V2 執行個體，升級已完成。 (通知方式之一是推出組態升級，這就是 2 階段升級)。現在，V2 執行個體可以讀取 V1 資料，將它轉換成 V2 資料、操作它，然後寫出為 V2 資料。 當其他執行個體讀取 V2 資料時，不需要轉換它，只要操作並寫出 V2 資料即可。

## <a name="next-steps"></a>後續步驟
若要了解建立向前相容的資料合約，請參閱 [向前相容資料合約](https://msdn.microsoft.com/library/ms731083.aspx)。

若要了解版本控制資料合約的最佳做法，請參閱 [資料合約版本控制](https://msdn.microsoft.com/library/ms731138.aspx)。

若要了解如何實作版本容錯的資料合約，請參閱 [版本相容序列化回呼](https://msdn.microsoft.com/library/ms733734.aspx)。

若要了解如何提供可跨多個版本相互操作的資料結構，請參閱 [IExtensibleDataObject](https://msdn.microsoft.com/library/system.runtime.serialization.iextensibledataobject.aspx)。
