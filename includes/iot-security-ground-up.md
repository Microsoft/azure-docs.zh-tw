---
title: 包含檔案
description: 包含檔案
services: iot-suite
author: dominicbetts
ms.service: iot-suite
ms.topic: include
ms.date: 04/24/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 181432c3050bbc614898b1ddf779bc90239a35be
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39189490"
---
# <a name="internet-of-things-security-from-the-ground-up"></a>徹底保護物聯網安全性

物聯網 (IoT) 使得全球企業面臨獨特的安全性、隱私權及相容性挑戰。 不同於傳統網路技術 (這類問題是以軟體及其實作方式為中心)，IoT 在意的是當網路與實體世界交會時會發生什麼事。 保護 IoT 解決方案要求確保安全佈建裝置，保護這些裝置與雲端之間的連接，以及在處理和儲存期間保護雲端中資料保護的安全。 但是，會針對這類功能運作的是資源受限的裝置、根據地理位置分佈的部署，以及解決方案中的大量裝置。

本文將說明 IoT 解決方案加速器如何提供安全且私密的物聯網雲端解決方案。 此解決方案加速器提供完整的端對端解決方案，徹底為每個階段內建安全性。 在 Microsoft，開發安全的軟體是軟體工程實務的一部分，這立基於 Microsoft 數十年來長時間開發安全軟體的體驗。 為了確保這一點，安全性開發週期 (SDL) 是基礎的開發方法，再加上基礎結構層級安全性服務的主機，例如營運安全性保證 (OSA)，以及 Microsoft 數位犯罪防治中心、Microsoft 安全性回應中心及 Microsoft 惡意程式碼防護中心。

此解決方案加速器提供獨特的功能，從 IoT 裝置佈建、連線及儲存資料都是簡單且明確的，而最棒的是很安全。 本文將檢驗 Azure IoT 解決方案加速器的安全性功能和部署策略，以確保能夠應付安全性、隱私權及相容性的挑戰。

## <a name="introduction"></a>簡介

物聯網 (IoT) 是未來的主流，可為企業提供即時且真實世界的商機，來降低成本、提高營收，並使其企業轉型。 不過，許多企業因為顧慮到安全性、隱私權及相容性問題，而對在組織中部署 IoT 而有所遲疑。 主要的考量點來自 IoT 基礎結構的獨特性，其會將網路與實體世界合併在一起，將這兩個世界中固有的個別風險混合。 IoT 的安全性是關於確保在裝置上執行的程式碼完整性、為裝置和使用者提供驗證、定義裝置 (以及這類裝置所產生的資料) 的明確擁有權，以及針對網路與實體攻擊進行復原。

接著，會有隱私權問題。 公司希望資料收集過程透明化，例如，要收集哪些資料及原因、可查看資料的人員、可控制存取的人員等。 最後，還有關於設備及操作人員的一般安全性問題，以及維護業界標準相容性的問題。

假如有安全性、隱私權、透明度及相容性考量，選擇正確的 IoT 解決方案提供者仍然是一項挑戰。 將由各種不同廠商所提供的 IoT 軟體和服務的各個部分聯結在一起，會難以偵測到安全性、隱私權、透明度及相容性中產生的隔閡，因此讓我們單獨進行修正。 選擇正確 IoT 軟體和服務提供者，是以尋找具有跨越多個縱向市場和地理位置執行的豐富經驗，同時還能以安全且透明的方式進行調整的提供者為基礎。 同樣地，它對於有數十年在全球無數部電腦上執行安全軟體之體驗的卓越提供者非常實用，並且能夠鑑別由這個物聯網的新世界所導致的威脅面。

## <a name="secure-infrastructure-from-the-ground-up"></a>徹底保護基礎結構的安全

[Microsoft Cloud](https://azure.microsoft.com) 基礎結構支援 127 個國家/地區中超過十億位客戶。 利用 Microsoft 數十年來建置企業軟體，以及在世界各地執行一些大型線上服務的經驗，相較於多數客戶可自行實現的部分，Microsoft Cloud 提供更高層級的增強安全性、隱私權、相容性及威脅減輕做法。

[安全性開發週期 (SDL)](https://www.microsoft.com/sdl/) 可提供必要的全公司開發程序，將安全性需求嵌入整個軟體週期。 為了協助確保營運活動會遵循相同層級的安全性做法，SDL 會使用 Microsoft 的營運安全性保證 (OSA) 程序中所列出的嚴謹安全性指導方針。 Microsoft 也會與第三方稽核公司合作，持續驗證其是否符合法律遵循義務，並且 Microsoft 會透過建立卓越的中心 (包括 Microsoft 數位犯罪防治中心、Microsoft 安全性回應中心及 Microsoft 惡意程式碼防護中心)，致力於產生廣泛的安全性成果。

## <a name="microsoft-azure---secure-iot-infrastructure-for-your-business"></a>Microsoft Azure - 適用於貴公司的安全 IoT 基礎結構

Microsoft Azure 提供完整的雲端解決方案，其中結合了持續成長的整合式雲端服務 (分析、機器學習服務、儲存體、安全性、網路功能及 Web) 集合，透過業界領先的承諾來為您的資料提供保護與隱私權。 Microsoft 的[模擬缺口](https://azure.microsoft.com/blog/red-teaming-using-cutting-edge-threat-simulation-to-harden-the-microsoft-enterprise-cloud/) 策略會透過由軟體安全性專家組成的專屬「紅隊」，來模擬攻擊、測試要偵測的 Azure 能力、防範新興威脅，以及從缺口中復原。 Microsoft 的[全域事件回應](https://www.microsoft.com/en-us/TrustCenter/Security/DesignOpSecurity)小組夜以繼日地工作，能夠減緩攻擊與惡意活動所產生的影響。 此小組會遵循事件管理、通訊及復原所建立的程序，並與內部和外部夥伴合作來使用可探索且可預測的介面。

Microsoft 的系統能提供持續的入侵偵測與防護、阻斷服務攻擊防護、一般滲透測試，以及可協助識別及緩解威脅的法務工具。 [Multi-Factor Authentication](../articles/active-directory/authentication/multi-factor-authentication.md) 可為存取網路的使用者提供額外的安全性層級。 此外，針對應用程式和主機提供者，Microsoft 會提供存取控制、監視、反惡意程式碼、弱點掃描、修補程式及組態管理。

此解決方案加速器會充分利用內建於 Azure 平台的安全性和隱私權，以及針對所有 Microsoft 軟體進行的安全開發，和作業所提供的 SDL 和 OSA 程序。 這些程序提供基礎結構保護、網路保護，以及身分識別與管理功能，以做為任何解決方案安全性的基礎。


  [IoT 解決方案加速器](../articles/iot-fundamentals/iot-introduction.md)內的 [Azure IoT 中樞](../articles/iot-hub/about-iot-hub.md)提供完全受控服務，使用每一裝置的安全性認證和存取控制，在 IoT 裝置與 Azure 服務之間啟用可靠且安全的雙向通訊，例如 [Azure Machine Learning](../articles/machine-learning/studio/what-is-machine-learning.md) 和 [Azure 串流分析](../articles/stream-analytics/stream-analytics-introduction.md)。

為了以最佳方式傳達內建於 Azure IoT 解決方案加速器的安全性和隱私權功能，本文將套件細分成三個主要的安全性領域。

![Azure IoT 解決方案加速器](media/iot-security-ground-up/securing-iot-ground-up-fig3.png)

### <a name="secure-device-provisioning-and-authentication"></a>保護裝置佈建和驗證

當裝置不在現場時，解決方案加速器會保護它們，方法是為每個裝置提供唯一的身分識別金鑰，在裝置運作時，IoT 基礎結構可用來與其進行通訊。 此程序非常快速且可輕易地設定。 利用使用者選取的裝置識別碼產生的金鑰會形成權杖的基礎，可以在裝置和 Azure IoT 中樞間的所有通訊中使用。

裝置識別碼可以在製造期間 (也就是硬體信任模組中所閃現的) 與裝置產生關聯，或是可以使用現有的固定識別碼作為 Proxy (例如 CPU 序號)。 由於變更裝置中的這個識別資訊並不簡單，因此請務必導入邏輯裝置識別碼，以防萬一基礎裝置硬體變更，但邏輯裝置維持不變。 在某些情況下，裝置身分識別的關聯會在裝置部署期間發生 (例如，經驗證的現場工程師會在與解決方案後端通訊的同時，實際設定新的裝置)。 [Azure IoT 中樞身分識別登錄](../articles/iot-hub/iot-hub-devguide.md) 會針對方案，為裝置身分識別和安全性金鑰提供安全的儲存體。 個別或群組的裝置身分識別可以新增到允許的清單或封鎖清單，以便完整控制裝置存取。

雲端中的 Azure IoT 中樞存取控制原則，能夠啟用和停用任何裝置身分識別，必要時可提供方法來取消關聯 IoT 部署中的裝置。 裝置的這個關聯和取消關聯是以每個裝置身分識別為根據。

其他裝置安全性功能包括︰

* 裝置不會接受未經要求的網訊連接。 它們會以僅限輸出的方式建立所有連接和路由。 若要讓裝置接收來自後端的命令，裝置必須初始連接以檢查是否有任何暫止的命令要處理。 在裝置和 IoT 中樞之間安全建立連接之後，在雲端和裝置之間來回傳遞訊息可以透明方式進行傳送。
* 裝置只會連接至或建立路由至與它們對等的知名服務，例如 Azure IoT 中樞。
* 系統層級的授權和驗證會使用每一裝置的身分識別，讓存取認證和權限能近乎即時撤銷。

### <a name="secure-connectivity"></a>安全的連線

訊息傳遞的持久性是所有 IoT 解決方案的重要功能。 永久傳遞命令和/或從裝置接收資料的需求，可透過下列事實強調說明：IoT 裝置是透過網際網路、或其他類似的不可靠網路來連線。 Azure IoT 中樞透過通知系統來提供雲端與裝置之間訊息傳遞的持久性，以回應訊息。 訊息傳遞的額外持久性可透過在 IoT 中樞快取訊息來達成，針對遙測最多七天，針對命令最多兩天。

為確保可在資源受限的環境中節省資源和作業，效率非常重要。 IoT 中樞支援 HTTPS (HTTP Secure) (熱門 http 通訊協定的業界標準安全版本)，能夠進行有效率的通訊。 Azure IoT 中樞支援的進階訊息佇列通訊協定 (AMQP) 和訊息佇列遙測傳輸 (MQTT)，不只是根據資源使用的效率而設計，同時也可進行可靠的訊息傳遞。 

延展性需要能夠與各式各樣裝置安全地交互操作的能力。 Azure IoT 中樞能夠安全連接至已啟用 IP 和未啟用 IP 的裝置。 已啟用 IP 的裝置能夠直接連接，並透過安全連接與 IoT 中樞進行通訊。 未啟用 IP 的裝置是資源受限的，只能透過短距離通訊協定 (例如 Zwave、ZigBee 和藍芽) 來連接。 現場閘道可用來彙總這些裝置，並執行通訊協定轉換，以便與雲端進行安全的雙向通訊。

其他連線安全性功能包括︰

* 裝置和 Azure IoT 中樞之間，或閘道和 Azure IoT 中樞之間的通訊路徑，會搭配使用 X.509 通訊協定驗證的 Azure IoT 中樞使用業界標準的傳輸層安全性 (TLS) 來保護。
* 若要保護裝置以防止來路不明的傳入連接，Azure IoT 中樞不會開啟任何裝置的連接。 裝置會起始所有連接。
* Azure IoT 中樞會永久儲存裝置的訊息，並等候要連接的裝置。 這些命令會儲存兩天，讓裝置可基於電力或連線能力的因素偶而進行連接來接收這些命令。 Azure IoT 中樞會維護每個裝置的每一裝置佇列。

### <a name="secure-processing-and-storage-in-the-cloud"></a>安全處理和雲端中的儲存體

透過加密通訊以處理雲端中的資料，解決方案加速器有助於確保資料安全。 這會提供彈性來實作額外加密並管理安全性金鑰。

使用 Azure Active Directory (AAD) 進行使用者驗證和授權，Azure IoT 解決方案加速器可以針對在雲端中的資料提供以原則為基礎的授權模型，啟用可稽核和檢閱的輕鬆存取管理。 此模型也能夠幾近即時的方式撤銷存取雲端中的資料，以及連到 Azure IoT 解決方案加速器的裝置。

一旦資料位於雲端之後，它就能進行處理並儲存於任何使用者定義的工作流程中。 存取資料的每個部分是根據所用的儲存體服務，透過 Azure Active Directory 來控制。

IoT 基礎結構所使用的所有金鑰都會儲存於雲端的安全儲存體中，並具備變換能力，以因應金鑰需要重新佈建的情況。 資料可以儲存在 [Azure Cosmos DB](../articles/cosmos-db/introduction.md) 或 [SQL Database](../articles/sql-database/sql-database-faq.md) 中，讓您能夠定義所需的安全性層級。 此外，Azure 提供一種方式來監視和稽核資料的所有存取，以提醒您有任何入侵或未經授權的存取。

## <a name="conclusion"></a>結論

物聯網是從您的事物開始 — 對企業最重要的事物。 IoT 可以藉由降低成本、提高營收，並使企業轉型，為企業提供令人讚嘆的價值。 這個轉型的成功，多半取決於選擇正確的 IoT 軟體和服務提供者。 這表示尋找提供者，其不僅可藉由了解企業需要與需求來催化這個轉型，也會提供使用安全性、隱私權、透明度及相容性建置的服務和軟體做為主要設計考量。 Microsoft 具備開發和部署安全軟體和服務的豐富經驗，並持續在這個新的物聯網紀元中保持領先的地位。

解決方案加速器根據設計會建置安全性措施，啟用安全的資產監視來改善效率、衍生營運效能來賦予創新，並採用進階的資料分析來使企業轉型。 解決方案加速器可利用關於安全性的分層式方式、多個安全性功能及設計模式，來協助部署基礎結構，可安心進行任何企業轉型。

## <a name="additional-information"></a>其他資訊

每個解決方案加速器都會建立 Azure 服務的執行個體，例如：

* [**Azure IoT 中樞**](https://azure.microsoft.com/services/iot-hub/)：將雲端連線到裝置的閘道。 您可以調整為每個中樞有百萬個連接，並利用每一裝置驗證支援來處理大量資料，以協助保護您的解決方案。
* [**Azure Cosmos DB**](https://azure.microsoft.com/services/cosmos-db/)：一個適用於半結構化資料的可調整、已完全編製索引的資料庫服務，可管理您所佈建裝置的中繼資料，例如屬性、組態及安全性屬性。 Azure Cosmos DB 提供高效能且高輸送量的處理、無從驗證結構描述的資料索引編製，以及豐富的 SQL 查詢介面。
* [**Azure 串流分析**](https://azure.microsoft.com/services/stream-analytics/)：雲端中處理的即時串流讓您能夠快速開發並部署低成本的分析方案，以在第一時間提供裝置、感應器、基礎結構與應用程式的深入剖析資料。 來自這個完全受控服務的資料可調整為任何數量，但仍可達到高輸送量、低遲性和恢復功能。
* [**Azure App Service**](https://azure.microsoft.com/services/app-service/)：一個雲端平台，可供建置功能強大的 Web 和行動應用程式來連接各地的資料；不論是在雲端還是內部部署環境內。 建置吸引客戶參與的 iOS、Android 和 Windows 版行動應用程式。 與軟體即服務 (SaaS) 和企業應用程式整合，讓您能夠立即連線到數十種雲端服務和企業應用程式。 使用您愛用的語言 (.NET、Node.JS、PHP、Python 或 Java) 和整合式開發環境 (IDE) 撰寫程式碼，以前所未有的速度建置 Web 應用程式和 API。
* [**Logic Apps**](https://azure.microsoft.com/services/app-service/logic/)：Azure App Service 的 Logic Apps 功能可協助您將 IoT 解決方案整合到現有的企業營運系統並自動化工作流程處理。 Logic Apps 可讓開發人員設計從觸發程序開始，然後執行一系列步驟的工作流程 — 使用功能強大的連接器來與您的商務程序整合的規則和動作。 Logic Apps 提供與 SaaS、雲端架構及內部部署應用程式的廣大生態系統的即時連接。
* [**Azure Blob 儲存體**](https://azure.microsoft.com/services/storage/)：可靠且符合經濟效益的雲端儲存體，適用於裝置要傳送到雲端的資料。