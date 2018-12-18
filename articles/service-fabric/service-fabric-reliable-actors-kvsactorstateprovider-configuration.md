---
title: 變更 Azure Service Fabric 動作項目中的 KVSActorStateProvider 設定 | Microsoft Docs
description: 了解設定 KVSActorStateProvider 類型的 Azure Service Fabric 可設定狀態的動作項目。
services: Service-Fabric
documentationcenter: .net
author: sumukhs
manager: timlt
editor: ''
ms.assetid: dbed72f4-dda5-4287-bd56-da492710cd96
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/2/2017
ms.author: sumukhs
ms.openlocfilehash: 81c09d61a5213319fa01ef5cc7070ffe385bbab1
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44049505"
---
# <a name="configuring-reliable-actors--kvsactorstateprovider"></a>設定 Reliable Actors - KVSActorStateProvider
您可以在指定之動作項目的 Config 資料夾下，變更 Microsoft Visual Studio 封裝根中所產生的 settings.xml，來修改 KVSActorStateProvider 的預設組態。

Azure Service Fabric 執行階段會在建立基礎執行階段元件時，在 settings.xml 檔案中尋找預先定義的區段名稱，並使用組態值。

> [!NOTE]
> 請 **不要** 刪除或修改在 Visual Studio 方案中產生之 settings.xml 檔案中的下列組態區段名稱。
> 
> 

## <a name="replicator-security-configuration"></a>複寫器安全性組態
複寫器安全性組態用來保護在複寫期間使用的通訊通道。 這表示服務將無法看到彼此的複寫流量，並且也會確保高度可用資料的安全。
依預設，空白的安全性組態區段會妨礙複寫安全性。

> [!IMPORTANT]
> 在 Linux 節點上，憑證必須是 PEM 格式。 若要深入了解如何尋找和設定適用於 Linux 的憑證，請參閱[在 Linux 上設定憑證](./service-fabric-configure-certificates-linux.md)。 
> 

### <a name="section-name"></a>區段名稱
&lt;ActorName&gt;ServiceReplicatorSecurityConfig

## <a name="replicator-configuration"></a>複寫器組態
複寫器組態會設定負責讓動作項目狀態提供者狀態高度可靠的複寫器。
預設組態由 Visual Studio 範本所產生，且應該已經足夠。 本節說明可用於微調複寫器的其他組態。

### <a name="section-name"></a>區段名稱
&lt;ActorName&gt;ServiceReplicatorConfig

### <a name="configuration-names"></a>組態名稱
| 名稱 | 單位 | 預設值 | 備註 |
| --- | --- | --- | --- |
| BatchAcknowledgementInterval |秒 |0.015 |次要複寫器收到作業後，將通知傳回給主要複寫器前所等待的時間間隔。 任何要在此間隔內傳送給作業處理的其他通知，會集中以一個回應傳送。 |
| ReplicatorEndpoint |N/A |無預設值--必要的參數 |主要/次要複寫器將用於與複本集中其他複寫器通訊的 IP 位址與連接埠。 這應該參考服務資訊清單中的 TCP 資源端點。 請參閱 [服務資訊清單資源](service-fabric-service-manifest-resources.md) ，深入了解如何在服務資訊清單中定義端點資源。 |
| RetryInterval |秒 |5 |複寫器若未收到作業通知，在重新傳輸訊息前的時間間隔。 |
| MaxReplicationMessageSize |位元組 |50 MB |單一訊息可傳輸的複寫資料大小上限。 |
| MaxPrimaryReplicationQueueSize |作業數目 |1024 |主要佇列中作業數目上限。 主要複寫器收到所有次要複寫器的通知後，系統便會釋放作業。 此值必須大於 64 且為 2 的乘冪。 |
| MaxSecondaryReplicationQueueSize |作業數目 |2048 |次要佇列中作業數目上限。 透過持續性讓狀態成為高可用性後，系統便會釋放作業。 此值必須大於 64 且為 2 的乘冪。 |

## <a name="store-configuration"></a>存放區組態
存放區組態用於設定本機存放區以用來保存正在複寫的狀態。
預設組態由 Visual Studio 範本所產生，且應該已經足夠。 本節將討論其他可用來微調本機存放區的組態。

### <a name="section-name"></a>區段名稱
&lt;ActorName&gt;ServiceLocalStoreConfig

### <a name="configuration-names"></a>組態名稱
| 名稱 | 單位 | 預設值 | 備註 |
| --- | --- | --- | --- |
| MaxAsyncCommitDelayInMilliseconds |毫秒 |200 |設定長期本機存放區認可的批次間隔上限。 |
| MaxVerPages |頁面數目 |16384 |本機存放區資料庫中版本頁面數上限。 其會判定未完成交易數上限。 |

## <a name="sample-configuration-file"></a>範例組態檔
```xml
<?xml version="1.0" encoding="utf-8"?>
<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Section Name="MyActorServiceReplicatorConfig">
      <Parameter Name="ReplicatorEndpoint" Value="MyActorServiceReplicatorEndpoint" />
      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
   </Section>
   <Section Name="MyActorServiceLocalStoreConfig">
      <Parameter Name="MaxVerPages" Value="8192" />
   </Section>
   <Section Name="MyActorServiceReplicatorSecurityConfig">
      <Parameter Name="CredentialType" Value="X509" />
      <Parameter Name="FindType" Value="FindByThumbprint" />
      <Parameter Name="FindValue" Value="9d c9 06 b1 69 dc 4f af fd 16 97 ac 78 1e 80 67 90 74 9d 2f" />
      <Parameter Name="StoreLocation" Value="LocalMachine" />
      <Parameter Name="StoreName" Value="My" />
      <Parameter Name="ProtectionLevel" Value="EncryptAndSign" />
      <Parameter Name="AllowedCommonNames" Value="My-Test-SAN1-Alice,My-Test-SAN1-Bob" />
   </Section>
</Settings>
```
## <a name="remarks"></a>備註
BatchAcknowledgementInterval 參數會控制複寫延遲性。 值為 '0' 時延遲可能性最低，但代價是降低輸送量 (隨著必須傳送與處理的通知訊息增加，每個訊息包含的通知會變少)。
BatchAcknowledgementInterval 的值越大，整體複寫輸送量越高，代價是作業延遲變高。 這會直接轉換成交易認可的延遲。

