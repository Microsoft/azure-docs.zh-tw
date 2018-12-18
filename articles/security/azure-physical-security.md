---
title: Azure 設施、廠房以及實體安全性 | Microsoft Docs
description: 本文說明 Azure 資料中心，包括實體基礎結構、安全性以及合規性供應項目。
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/28/2018
ms.author: terrylan
ms.openlocfilehash: 69a54640ae3b4e71d7782712ad1764babab7104a
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39170672"
---
# <a name="azure-facilities-premises-and-physical-security"></a>Azure 設施、廠房以及實體安全性
Azure 是由[分散在世界各地的資料中心基礎結構](https://azure.microsoft.com/global-infrastructure/)所組成，可支援支援數千項線上服務，並且橫跨全球超過 100 個高度安全的設施。

此基礎結構的設計訴求是將應用程式放在最靠近全球不同地區使用者的地點、預留資料落地，並為客戶提供完整的合規性與存放區選項。 Azure 在全球有 52 個區域，並且可在 140 個國家/地區使用。

區域是一組透過大規模且富有彈性的網路互連的資料中心。 依預設，此網路包含內容散發、負載平衡、備援及加密。 因為 Azure 提供的全球區域多於其他任何雲端提供者，可以讓您進行更彈性的規劃，將應用程式部署在需要的位置。

Azure 區域會依地理區排列。 Azure 地理可確保符合地理界限內的資料落地、主權、合規性及復原需求。

不同的地理區讓具有特殊資料落地與合規性需求的客戶，可以將他們的資料與應用程式存放在彼此鄰近的區域。 因為每個地理區都連線到專用的高容量網路基礎結構，所以都具備容錯能力，禁得起整個區域失敗。

可用性區域與 Azure 區域實際上是兩個各自獨立的地點。 每個可用性區域由一或多個資料中心組成，配備了電力、冷卻系統及網路系統。 可用性區域的可用性與低延遲複寫特性，可以讓您執行任務關鍵性的應用程式。

下圖顯示 Azure 全域基礎結構如何將相同資料落地界限內的區域與可用性區域配對，以供高可用性、災害復原及備份之用。

![顯示資料落地界限的圖表][1]

分散各地的資料中心讓 Microsoft 得以接近客戶，進而減少網路延遲，並容許進行異地備援備份和容錯移轉。

## <a name="physical-security"></a>實體安全性
Microsoft 會經由嚴格控制您資料儲存所在區域的實體存取，來設計、建置及操作資料中心。 Microsoft 了解保護您資料的重要性，並致力於協助保護內含資料的資料中心。 Microsoft 有一整個部門專門負責設計、建置及操作可支援 Azure 的實體設施。 這個團隊致力於維護最新的實體安全性。

Microsoft 採用分層式實體安全性方法，以降低未經授權的使用者取得資料和資料中心資源實體存取權的風險。 由 Microsoft 管理的資料中心有廣泛的保護層級：在設施的周邊、建築物的周邊、建築物內部，以及在資料中心樓層核准存取權。 實體安全性的層級包括：

- **存取要求與核准。** 您必須在抵達資料中心之前，要求存取權。 您必須針對您的造訪提供正當的業務理由，例如合規性或稽核用途。 Microsoft 員工會根據需要存取原則來核准所有要求。 需要存取原則有助於讓需要在資料中心完成工作的個人數目保持在最低限度。 Microsoft 授與權限後，根據核准的業務理由，個人只擁有必要資料中心離散區域的存取權。 權限受限於特定一段時間，超過這段時間之後就會過期。

- **設施周邊。** 當您抵達資料中心時，必須通過妥善定義的存取點。 一般而言，鋼筋水泥構成的高欄包圍了周邊的每一英吋。 資料中心四周都有攝影機，安全性小組會全天候監視這些影片。

- **建築物入口。** 資料中心入口配置了經過嚴格訓練和背景調查的專業保全人員。 這些保全人員也會定期巡邏資料中心，他們也會全天候監視資料中心內的攝影機影片。

- **建築物內部。** 進入建築物之後，您必須通過採用生物識別技術的雙因素驗證，才能繼續在資料中心內移動。 如果身分識別經過驗證，您只可以進入已獲准存取的資料中心部分。 只有在核准的持續時間內，您才可以停留在那裡。

- **資料中心樓層。** 您只能夠進入獲准進入的樓層。 您必須通過全身金屬探測檢查。 為了降低在我們不知情的情況下，資料未經授權進入或離開資料中心的風險，只有經過核准的裝置可以進入資料中心樓層。 此外，攝影機會監視每個伺服器機櫃的前方與後方。 當您離開資料中心樓層時，必須再次通過全身金屬探測檢查。 若要離開資料中心，您必須通過其他安全掃描。

Microsoft 要求訪客從任何 Microsoft 設施離開時須交回徽章。

## <a name="physical-security-reviews"></a>安全性審查
我們會定期執行設施的實體安全性審查，以確保資料中心適當地滿足 Azure 安全性需求。 資料中心主機服務提供者人員不會提供 Azure 服務管理。 這些人員無法登入 Azure 系統或沒有 Azure 共置空間和網架的實體存取權。

## <a name="data-bearing-devices"></a>資料軸承裝置
Microsoft 會使用最佳做法程序以及[符合 NIST 800-88 規範](https://csrc.nist.gov/publications/detail/sp/800-88/archive/2006-09-01)的清除解決方案。 對於無法清除的硬碟，我們會使用可將它們銷毀，讓資訊不可能復原的銷毀程序。 此銷毀程序可能是弄碎、切碎、粉碎或焚化。 我們會根據資產類型決定處理方法。 並且保留銷毀的記錄。  

## <a name="equipment-disposal"></a>設備處置
在系統的生命週期結束時，Microsoft 作業人員會遵循嚴格的資料處理和硬體處置程序，確保不受信任的對象無法存取可能包含您資料的硬體。 我們會對硬體使用其支援的安全清除方法。 對於無法清除的硬碟，我們會使用可將該硬碟銷毀，讓資訊不可能復原的銷毀程序。 此銷毀程序可能是弄碎、切碎、粉碎或焚化。 我們會根據資產類型決定處理方法。 並且保留銷毀的記錄。 所有 Azure 服務都會使用已核准的媒體儲存體和處置管理服務。

## <a name="compliance"></a>法規遵循
我們會設計及管理 Azure 基礎結構，以符合一組廣泛的國際性及產業特定合規性標準，例如 ISO 27001、HIPAA、FedRAMP、SOC 1 和 SOC 2。 也可符合國家特定標準，包括澳洲 IRAP、英國 G-Cloud 及新加坡 MTCS。 嚴格的第三方稽核，例如「英國標準學會」所做的稽核，以確認符合這些標準要求的嚴格安全性控制。

如需 Azure 所遵守的合規性標準完整清單，請參閱[合規性供應項目](https://www.microsoft.com/trustcenter/compliance/complianceofferings)。 

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為協助保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件和界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)

<!--Image references-->
[1]: ./media/azure-physical-security/data-residency-boundary.png
