---
title: Azure Service Fabric 中的 Reliable Collection 物件序列化 | Microsoft Docs
description: Azure Service Fabric Reliable Collections 物件序列化
services: service-fabric
documentationcenter: .net
author: mcoskun
manager: timlt
editor: masnider,rajak
ms.assetid: 9d35374c-2d75-4856-b776-e59284641956
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 5/8/2017
ms.author: mcoskun
ms.openlocfilehash: 8fb6f1767741e950b300fd297250a6b64656191c
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37952421"
---
# <a name="reliable-collection-object-serialization-in-azure-service-fabric"></a>Azure Service Fabric 中的 Reliable Collection 物件序列化
Reliable Collections 會複寫並保存其項目，以確保在電腦發生失敗和電力中斷時能持續保留這些項目。
不論是要複寫還是保存項目，Reliable Collections 都必須將它們序列化。

Reliable Collections 會從 Reliable State Manager 為指定的類型取得適當的序列化程式。
Reliable State Manager 包含內建的序列化程式，並允許為指定的類型註冊自訂序列化程式。

## <a name="built-in-serializers"></a>內建的序列化程式

Reliable State Manager 包含一些常見類型的內建序列化程式，以便透過預設方式有效率地將這些類型序列化。 針對其他類型，Reliable State Manager 會回復為使用 [DataContractSerializer](https://msdn.microsoft.com/library/system.runtime.serialization.datacontractserializer(v=vs.110).aspx)。
內建的序列化程式較有效率，因為它們知道它們的類型無法變更，所以不需要包含類型的相關資訊 (例如其類型名稱)。

Reliable State Manager 具有下列類型的內建序列化程式： 
- Guid
- 布林
- byte
- sbyte
- byte[]
- char
- 字串
- decimal
- double
- float
- int
- uint
- long
- ulong
- short
- ushort

## <a name="custom-serialization"></a>自訂序列化

自訂序列化程式通常是用來提升效能，或是透過網路和在磁碟上加密資料。 自訂序列化程式通常比一般序列化程式更有效率，最主要的原因是因為它們不需要將類型的相關資訊序列化。 

[IReliableStateManager.TryAddStateSerializer<T>](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.data.ireliablestatemanager.tryaddstateserializer) 可用來為指定的類型 T 註冊自訂序列化程式。您應該在建構 StatefulServiceBase 時進行這項註冊，以確保在開始復原之前，所有 Reliable Collections 都能存取相關的序列化程式以讀取其保存資料。

```csharp
public StatefulBackendService(StatefulServiceContext context)
  : base(context)
  {
    if (!this.StateManager.TryAddStateSerializer(new OrderKeySerializer()))
    {
      throw new InvalidOperationException("Failed to set OrderKey custom serializer");
    }
  }
```

> [!NOTE]
> 自訂序列化程式的優先順序高於內建的序列化程式。 例如，在為 int 註冊自訂序列化程式之後，系統就會使用它將整數序列化，而不會使用 int 的內建序列化程式。

### <a name="how-to-implement-a-custom-serializer"></a>如何實作自訂序列化程式

自訂序列化程式需要實作 [IStateSerializer<T>](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.data.istateserializer-1) 介面。

> [!NOTE]
> IStateSerializer<T> 包含會接受額外 T (稱為基底值) 的 Write 和 Read 多載。 此 API 適用於差異序列化。 目前並未公開差異序列化功能。 因此，將等到公開並啟用差異序列化之後，才會呼叫這兩個多載。

以下是一個名為 OrderKey 且包含四個屬性的範例自訂類型

```csharp
public class OrderKey : IComparable<OrderKey>, IEquatable<OrderKey>
{
    public byte Warehouse { get; set; }

    public short District { get; set; }

    public int Customer { get; set; }

    public long Order { get; set; }

    #region Object Overrides for GetHashCode, CompareTo and Equals
    #endregion
}
```

以下是 IStateSerializer<OrderKey> 的範例實作。
請注意，接受 baseValue 的 Read 和 Write 多載會呼叫其個別的多載以滿足向後相容性。

```csharp
public class OrderKeySerializer : IStateSerializer<OrderKey>
{
  OrderKey IStateSerializer<OrderKey>.Read(BinaryReader reader)
  {
      var value = new OrderKey();
      value.Warehouse = reader.ReadByte();
      value.District = reader.ReadInt16();
      value.Customer = reader.ReadInt32();
      value.Order = reader.ReadInt64();

      return value;
  }

  void IStateSerializer<OrderKey>.Write(OrderKey value, BinaryWriter writer)
  {
      writer.Write(value.Warehouse);
      writer.Write(value.District);
      writer.Write(value.Customer);
      writer.Write(value.Order);
  }
  
  // Read overload for differential de-serialization
  OrderKey IStateSerializer<OrderKey>.Read(OrderKey baseValue, BinaryReader reader)
  {
      return ((IStateSerializer<OrderKey>)this).Read(reader);
  }

  // Write overload for differential serialization
  void IStateSerializer<OrderKey>.Write(OrderKey baseValue, OrderKey newValue, BinaryWriter writer)
  {
      ((IStateSerializer<OrderKey>)this).Write(newValue, writer);
  }
}
```

## <a name="upgradability"></a>可升級性
在 [輪流應用程式升級](service-fabric-application-upgrade.md)中，升級會套用至節點的子集，一次一個升級網域。 在此過程中，某些升級網域會比您的應用程式版本新，而某些升級網域會比您的應用程式的版本舊。 在首度發行期間，新版的應用程式必須能夠讀取舊版的資料，而舊版的應用程式必須能夠讀取新版的資料。 如果資料格式沒有向前及向後相容，升級便可能會失敗，或是發生更糟糕的狀況，像是資料可能會遺失或損毀。

如果您使用的是內建序列化程式，就不需要擔心相容性。
不過，如果您使用的是自訂序列化程式或 DataContractSerializer，資料就必須無限地往前和往後相容。
換句話說，每個還原序列化版本都必須能夠將該類型的任何版本序列化或還原序列化。

「資料合約」使用者應該依照妥善定義的版本控制規則來新增、移除及變更欄位。 「資料合約」也支援處理未知欄位、連結到序列化和還原序列化程序，以及處理類別繼承。 如需詳細資訊，請參閱 [使用資料合約](https://msdn.microsoft.com/library/ms733127.aspx)。

自訂序列化程式使用者應該遵守所使用序列化程式的準則，以確保能夠往前和往後相容。
支援所有版本的常見方式是在開頭新增大小資訊，並且只新增選用的屬性。
如此一來，每個版本便可儘可能地讀取資料，然後跳過資料流的其餘部分。

## <a name="next-steps"></a>後續步驟
  * [序列化與升級](service-fabric-application-upgrade-data-serialization.md)
  * [可靠的集合的開發人員參考資料](https://msdn.microsoft.com/library/azure/microsoft.servicefabric.data.collections.aspx)
  * [使用 Visual Studio 升級您的應用程式](service-fabric-application-upgrade-tutorial.md) 將引導您完成使用 Visual Studio 進行應用程式升級的步驟。
  * [使用 PowerShell 升級您的應用程式](service-fabric-application-upgrade-tutorial-powershell.md) 將引導您完成使用 PowerShell 進行應用程式升級的步驟。
  * 使用 [升級參數](service-fabric-application-upgrade-parameters.md)來控制您應用程式的升級方式。
  * 參考 [進階主題](service-fabric-application-upgrade-advanced.md)，以了解如何在升級您的應用程式時使用進階功能。
  * 參考 [疑難排解應用程式升級](service-fabric-application-upgrade-troubleshooting.md)中的步驟，以修正應用程式升級中常見的問題。
