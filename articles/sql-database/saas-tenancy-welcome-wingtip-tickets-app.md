---
title: 歡迎使用 Wingtips 應用程式 - Azure SQL Database | Microsoft Docs
description: 了解雲端環境中 Azure SQL Database 的資料庫租用模型和相關範例 Wingtips SaaS 應用程式。
keywords: SQL Database Azure
services: sql-database
author: billgib
manager: craigg
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: billgib
ms.openlocfilehash: 3c7d1d40af3a0b8f70302171eb13ac0a180b0bfe
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34644387"
---
# <a name="the-wingtip-tickets-saas-application"></a>Wingtip Tickets SaaS 應用程式

相同的 *Wingtip Tickets* SaaS 應用程式會分別在三個範例中實作。 此應用程式是以小型場地 (戲院、俱樂部等) 為目標的簡易活動列表與售票 SaaS 應用程式。每個場地都是應用程式的租用戶，且都有其本身的資料：場地詳細資料、活動清單、客戶、訂票資料等。此應用程式可搭配使用管理指令碼和教學課程，示範端對端的 SaaS 案例。 其中包括佈建租用戶、監視及管理效能、結構描述管理，以及跨租用戶報告和分析。

## <a name="three-saas-application-and-tenancy-patterns"></a>三個 SaaS 應用程式和租用模式

應用程式有三個可用的版本；每個版本分別會在 Azure SQL Database 上探索不同的資料庫租用模式。  第一個模式會使用每一租用戶一個獨立應用程式模式搭配它自己的資料庫。 第二個模式會使用每個租用戶各有一個資料庫的多租用戶應用程式。 第三個範例會使用具有分區化多租用戶資料庫的多租用戶應用程式。

![三種租用模式][image-three-tenancy-patterns]

 每個範例都包含應用程式程式碼，以及管理指令碼，和會探索特定範圍的設計與管理模式的教學課程。  每個範例都可在五分鐘內完成部署。  這三個範例可用並存的方式部署，以便您比較設計和管理上的差異。

## <a name="standalone-application-per-tenant-pattern"></a>每一租用戶一個獨立應用程式模式

每一租用戶一個獨立應用程式模式會使用每個租用戶分別有一個資料庫的單一租用戶應用程式。 每個租用戶的應用程式 (包含其資料庫) 會部署在個別的 Azure 資源群組中。 此資源群組可以部署在服務提供者的訂用帳戶或租用戶的訂用帳戶中，並且由提供者代表租用戶加以管理。 每一租用戶一個獨立應用程式模式會提供最大的租用戶隔離性，但通常最耗費資源，因為無法在多個租用戶之間共用資源。  此模式相當適合可能更複雜、且部署至較少租用戶的應用程式。  透過獨立部署，相較於其他模式，可更輕鬆地針對每個租用戶自訂應用程式。  

請參閱[教學課程][docs-tutorials-for-wingtip-sa]和 GitHub [.../Microsoft/WingtipTicketsSaaS-StandaloneApp][github-code-for-wingtip-sa] 上的程式碼。

## <a name="database-per-tenant-pattern"></a>每一租用戶一個資料庫的模式

對於注重租用戶隔離性，並且想要執行集中式服務以有效運用共用資源而發揮成本效益的服務提供者，使用「每一租用戶一個資料庫的模式」將有其效用。 針對每個場所或租用戶分別會建立一個資料庫，且所有資料庫會集中受到管理。 資料庫可裝載於彈性集區中，以提供符合成本效益且便利的管理程序，而妥善因應租用戶無法預期的工作負載模式。 目錄資料庫會包含租用戶和其資料庫之間的對應。 此對應可使用能夠對應用程式進行有效連線管理的[彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md)的分區對應管理功能來管理。

請參閱[教學課程][docs-tutorials-for-wingtip-dpt]和 GitHub [.../Microsoft/WingtipTicketsSaaS-DbPerTenant][github-code-for-wingtip-dpt] 上的程式碼。

## <a name="sharded-multi-tenant-database-pattern"></a>分區化多租用戶資料庫模式

對於想要降低每一租用戶的成本，並且可承受較低租用戶隔離性的服務提供者，多租用戶資料庫將有其效用。 此模式可將大量租用戶封裝到單一資料庫中，而降低每一租用戶的成本。 藉由將多個租用戶分區化到多個資料庫間，將可達到近乎無限大的規模。 目錄資料庫會將租用戶對應到資料庫。  

此模式也可支援混合模型，讓您可透過在一個資料庫中包含多個租用戶的方式進行成本最佳化，或透過每一租用戶各有其資料庫的方式進行隔離性最佳化。 您可以在租用戶佈建時或是稍後，就個別的租用戶進行這項選擇，而不會對應用程式造成影響。  必須以不同方式處理租用戶的群組時，可以有效地使用此模型。 例如，可以將低成本的租用戶指派給共用資料庫，而將進階租用戶指派給他們自己的資料庫。 

請參閱[教學課程][docs-tutorials-for-wingtip-mt]和 GitHub [.../Microsoft/WingtipTicketsSaaS-MultiTenantDb][github-code-for-wingtip-mt] 上的程式碼。

## <a name="next-steps"></a>後續步驟

#### <a name="conceptual-descriptions"></a>概念說明

- 如需應用程式租用模式的詳細說明，請參閱[多租用戶 SaaS 資料庫租用模式][saas-tenancy-app-design-patterns-md]

#### <a name="tutorials-and-code"></a>教學課程和程式碼

- 每個租用戶獨立應用程式：
    - [獨立應用程式的教學課程][docs-tutorials-for-wingtip-sa]。
    - [獨立應用程式的程式碼，位於 GitHub][github-code-for-wingtip-sa]。

- 每一租用戶一個資料庫：
    - [每一租用戶一個資料庫的教學課程][docs-tutorials-for-wingtip-dpt]。
    - [每一租用戶一個資料庫的程式碼，位於 GitHub][github-code-for-wingtip-dpt]。

- 分區化多租用戶：
    - [分區化多租用戶的教學課程][docs-tutorials-for-wingtip-mt]。
    - [分區化多租用戶的程式碼，位於 GitHub][github-code-for-wingtip-mt]。



<!-- Image references. -->

[image-three-tenancy-patterns]: media/saas-tenancy-welcome-wingtip-tickets-app/three-tenancy-patterns.png "三種租用模式。"

<!-- Docs.ms.com references. -->

[saas-tenancy-app-design-patterns-md]: saas-tenancy-app-design-patterns.md

<!-- WWWeb http references. -->

[docs-tutorials-for-wingtip-sa]: https://aka.ms/wingtipticketssaas-sa
[github-code-for-wingtip-sa]: https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp

[docs-tutorials-for-wingtip-dpt]: https://aka.ms/wingtipticketssaas-dpt
[github-code-for-wingtip-dpt]: https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant

[docs-tutorials-for-wingtip-mt]: https://aka.ms/wingtipticketssaas-mt
[github-code-for-wingtip-mt]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDb

