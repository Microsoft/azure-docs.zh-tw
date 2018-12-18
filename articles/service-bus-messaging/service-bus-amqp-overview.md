---
title: Azure 服務匯流排的 AMQP 1.0 概觀 | Microsoft Docs
description: 了解如何在 Azure 中使用 Advanced Message Queuing Protocol (AMQP)。
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 0e8d19cc-de36-478e-84ae-e089bbc2d515
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 01/30/2018
ms.author: spelluru
ms.openlocfilehash: 8fd87897a70b0a7bb629e185473374837cf1026e
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701929"
---
# <a name="amqp-10-support-in-service-bus"></a>服務匯流排中的 AMQP 1.0 支援
Azure 服務匯流排雲端服務和內部部署 [Service Bus for Windows Server (Service Bus 1.1)](https://msdn.microsoft.com/library/dn282144.aspx) 兩者都支援進階訊息佇列通訊協定 (AMQP) 1.0。 透過開放式標準通訊協定，AMQP 可讓您打造一個跨平台的混合式應用程式。 您可以透過使用不同語言和架構所建立，且在不同作業系統上執行的元件來建構應用程式。 所有這些元件都可以連線到服務匯流排，並有效地且不失真地順暢交換結構化的商業訊息。

## <a name="introduction-what-is-amqp-10-and-why-is-it-important"></a>簡介：何謂 AMQP 1.0 以及它為什麼很重要？
傳統上，訊息導向的中介軟體產品會採用專屬的通訊協定，以進行用戶端應用程式和代理程式之間的通訊。 這表示一旦選取特定的廠商訊息代理程式，您就必須使用該廠商的程式庫來連接用戶端應用程式與該代理程式。 結果便是您會一定程度地與該廠商相依，因為若要將應用程式移植到其他產品，將需要變更所有已連接應用程式的程式碼。 

此外，從不同廠商連接訊息代理程式有些麻煩。 通常需要應用程式層級的橋接功能，才能在不同系統間移動訊息，以及轉換成這些系統專屬的訊息格式。 舉例來說，在將新的統一介面提供給較舊的不同系統使用，或在合併公司之後欲整合 IT 系統時，便常常有這種需求。

軟體產業是門瞬息萬變的生意；有時還沒回應過來便又出現了新的程式設計語言和應用程式架構。 同樣的，IT 系統的需求也會隨著時間而進化，因此開發人員會想要充分利用最新的平台功能。 不過，選取的訊息廠商有時並不支援這些平台。 因為訊息通訊協定是專用的，所以其他人無法為這些新平台提供程式庫。 因此，您必須使用建置閘道或橋接器等方法，讓您得以繼續使用傳訊產品。

開發 進階訊息佇列通訊協定 (AMQP) 1.0 的動機就是為了處理這些問題。 AMQP 是 JP Morgan Chase 的發明，與大多數的金融服務公司一樣，JP Morgan Chase 是訊息導向中介軟體的重度使用者。 目的很簡單：建立一個開放式標準訊息通訊協定，讓透過使用不同語言、架構及作業系統所建置的元件能夠建置訊息架構應用程式，並且全都使用一組供應商所提供的最優質元件。

## <a name="amqp-10-technical-features"></a>AMQP 1.0 技術功能
AMQP 1.0 是一個有效率且可靠的有線等級訊息通訊協定，可以用來建置強大的跨平台訊息應用程式。 此通訊協定有一個簡單目的︰就是在兩個用戶端間定義安全、可靠且有效率的訊息傳輸機制。 訊息本身是使用可攜式資料表示法來編碼，讓異質傳送者和接收者可透過百分之百臨場感來交換結構化的商業訊息。 下列清單摘要說明最重要的功能：

* **效率**：AMQP 1.0 是連線導向的通訊協定，可針對通訊協定指示和透過其傳輸的商業訊息使用二進位編碼。 其中包含精密的流程控制配置，可以將網路與連接的元件使用率最大化。 也就是說，此通訊協定的設計目的是在效率、彈性及交互操作性之間取得平衡。
* **可靠**：AMQP 1.0 通訊協定讓您能夠利用某個範圍的可靠性保證來交換訊息，從「射後不理」變成可靠且只認可一次的傳遞。
* **彈性**：AMQP 1.0 是一種有彈性的通訊協定，讓您可以用來支援不同的拓撲。 您可以使用相同的通訊協定，來進行用戶端對用戶端、用戶端對代理人的通訊，以及代理人對代理人的通訊。
* **與訊息代理程式模型無關**：AMQP 1.0 規格不會在訊息代理程式使用的訊息模型上強制執行任何需求。 這表示您能夠將 AMQP 1.0 支援新增至現有的訊息代理人。

## <a name="amqp-10-is-a-standard-with-a-capital-s"></a>AMQP 1.0 是一項標準 (Standard 的 S 為大寫)
AMQP 1.0 是由 ISO 與 IEC 核定為 ISO/IEC 19464:2014 的國際標準。

有一個由包含技術提供者及使用者公司在內超過 20 家公司所組成的核心群組，從 2008 年起就開始開發 AMQP 1.0。 在這段期間，使用者公司貢獻了他們的實際商業需求，而技術廠商逐步開發出此通訊協定來符合這些需求。 在整個過程中，廠商參與了他們共同合作的研討會，一起驗證他們實作間的交互操作性。

在 2011 年 10 月，開發工作移轉到資訊結構標準發展組織 (OASIS) 內的一個技術委員會，並在 2012 年 10 月發佈 OASIS AMQP 1.0 標準。 以下為在標準開發期間參與了該技術委員會的公司：

* **技術廠商**：Axway Software、Huawei Technologies、IIT Software、INETCO Systems、Kaazing、Microsoft、Mitre Corporation、Primeton Technologies、Progress Software、Red Hat、SITA、Software AG、Solace Systems、VMware、WSO2、Zenika。
* **使用者公司**：Bank of America、Credit Suisse、Deutsche Boerse、Goldman Sachs、JPMorgan Chase。

以下是一些關於開放標準之最常引用的優點：

* 比較不會被廠商鎖定
* 互通性
* 可以廣泛使用程式庫與工具
* 提供保護以免過時
* 對於有經驗之員工的可用性
* 較低且可管理的風險

## <a name="amqp-10-and-service-bus"></a>AMQP 1.0 和服務匯流排
Azure 服務匯流排中的 AMQP 1.0 支援代表您現在能夠從一組平台中使用有效率的二進位通訊協定，來運用服務匯流排佇列與發佈/訂閱代理訊息功能。 此外，您還可以建置使用混合語言、架構及作業系統所建置之元件所組成的應用程式。

下圖說明的是一個部署範例，其中的 Java 用戶端是在 Linux 上執行並使用標準的 Java 訊息服務 (JMS) API 撰寫而成，而 .NET 用戶端則是在 Windows 上執行並使用 AMQP 1.0 透過服務匯流排來交換訊息。

![][0]

**圖 1：範例部署案例示範使用服務匯流排和 AMQP 1.0 的跨平台訊息服務**

目前已知下列用戶端程式庫可以使用服務匯流排：

| 語言 | 程式庫 |
| --- | --- |
| Java |Apache Qpid Java 訊息服務 (JMS) 用戶端<br/>IIT Software SwiftMQ Java 用戶端 |
| C |Apache Qpid Proton-C |
| PHP |Apache Qpid Proton-PHP |
| Python |Apache Qpid Proton-Python |
| C# |AMQP .Net Lite |

**圖 2：AMQP 1.0 用戶端程式庫的資料表**

## <a name="summary"></a>總結
* AMQP 1.0 是一個既開放又可靠的訊息通訊協定，可以用來建置跨平台的混合應用程式。 AMQP 1.0 是一項 OASIS 標準。
* Azure 服務匯流排和 Service Bus for Windows Server (Service Bus 1.1) 中現可支援 AMQP 1.0。 價格與現有的通訊協定一樣。

## <a name="next-steps"></a>後續步驟
準備好進行深入了解嗎？ 請造訪下列連結：

* [搭配使用 .NET 的服務匯流排與 AMQP]
* [搭配使用 Java 的服務匯流排與 AMQP]
* [在 Azure Linux VM 上安裝 Apache Qpid Proton-C]
* [Windows Server 服務匯流排中的 AMQP]

[0]: ./media/service-bus-amqp-overview/service-bus-amqp-1.png
[搭配使用 .NET 的服務匯流排與 AMQP]: service-bus-amqp-dotnet.md
[搭配使用 Java 的服務匯流排與 AMQP]: service-bus-amqp-java.md
[在 Azure Linux VM 上安裝 Apache Qpid Proton-C]: service-bus-amqp-apache.md
[Windows Server 服務匯流排中的 AMQP]: https://msdn.microsoft.com/library/dn574799.aspx
