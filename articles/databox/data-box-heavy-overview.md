---
title: Microsoft Azure Data Box Heavy 概觀 | Microsoft Docs
description: 說明 Azure 資料箱，這是可讓您將大量資料傳輸至 Azure 的混合式解決方案
services: databox
documentationcenter: NA
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: databox
ms.devlang: NA
ms.topic: overview
ms.custom: ''
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 09/24/2018
ms.author: alkohli
ms.openlocfilehash: 8ecef86841bcf13a469f9c0dc81f114bd54acdba
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946508"
---
# <a name="what-is-azure-data-box-heavy-preview"></a>什麼是 Azure Data Box Heavy？ (預覽)

Microsoft Azure 資料箱混合式解決方案可讓您以快速、實惠且可靠的方式，將數百 TB 的內部部署資料傳送至 Azure。 我們會透過貨運向您寄送專屬的 1 PB 容量存放裝置，來加速安全的資料傳輸。 該裝置有堅固的外殼，可在運輸過程中保護裝置與資料。

Data Box Heavy 目前為預覽狀態，您可以透過 Azure 入口網站註冊以申請裝置。 當您的資料中心收到裝置之後，您便可以使用本機 Web UI 來設定它。 將您伺服器的資料複製到裝置上，然後將裝置寄回給 Azure。 在 Azure 資料中心內，您的資料會自動從裝置上傳到 Azure。 Azure 入口網站中的資料箱服務會以端對端的方式追蹤這整個程序。


> [!IMPORTANT]
> - Data Box Heavy 目前為預覽狀態。 部署這個解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 
> - 若要申請裝置，請在[預覽入口網站](http://aka.ms/)中註冊。
> - 在預覽期間，Data Box Heavy 可運送給美國與歐盟的客戶。 如需詳細資訊，請移至[區域可用性](#region-availability)。

## <a name="use-cases"></a>使用案例

Data Box Heavy 非常適合在網路連線能力受限或沒有網路連線能力的情況下，傳輸大於 500 TB 的資料。 資料移動可以是單次或定期的，或一開始是大量資料傳輸，然後繼之以定期傳輸。 以下是適合將 Data Box Heavy 用於資料傳輸的各種案例。

 - **單次移轉** - 有大量的內部部署資料移至 Azure 時。 
     - 將媒體櫃從離線磁帶移至 Azure 以建立線上媒體櫃。
     - 將 VM 陣列、SQL 伺服器與應用程式移轉到 Azure
     - 將歷程記錄資料移至 Azure 以使用 HDInsight 進行深入分析和報告

 - **初始大量傳輸** - 使用 Data Box Heavy (種子) 完成初始大量傳輸，然後透過網路進行累加式傳輸。 
     - 例如，使用 Commvault 等備份解決方案合作夥伴與 Data Box Heavy 來將初始的大量歷程記錄備份移到 Azure。 完成後，繼續透過網路將累加資料傳輸到 Azure 儲存體。

 - **定期上傳** - 定期產生大量資料而需要移至 Azure 時。 例如，在進行能源探勘時，鑽井平台和風車農場會產生視訊內容。      

## <a name="benefits"></a>優點

Data Box Heavy 的設計可讓它在幾乎不對網路產生影響的情況下，將大量資料移至 Azure。 此解決方案有下列優點：

- **速度** - Data Box Heavy 使用的是高效能的 40 Gbps 網路介面。

- **安全** - Data Box Heavy 具有適用於裝置、資料與服務的內建安全性保護機制。
    - 裝置有堅固的裝置外殼，並以防拆螺絲和易碎貼紙加以保護。 
    - 裝置上的資料會全程受到 AES 256 位元加密的保護。
    - 裝置只能使用 Azure 入口網站中提供的密碼來解除鎖定。
    - 此服務受到 Azure 安全功能的保護。
    - 在您的資料上傳至 Azure 後，裝置上的磁碟即會根據 NIST 800-88r1 標準進行抹除。


<!--## Features and specifications

The Data Box Heavy device has the following features in this release.

| Specifications                                          | Description              |
|---------------------------------------------------------|--------------------------|
| Weight                                                  | < 50 lbs.                |
| Dimensions                                              | Device - Width: 309.0 mm Height: 430.4 mm Depth: 502.0 mm |            
| Rack space                                              | 7 U when placed in the rack on its side (cannot be rack-mounted)|
| Cables required                                         | 1 X power cable (included) <br> 2 RJ45 cables <br> 2 X SFP+ Twinax copper cables|
| Storage capacity                                        | 100 TB <br> 80 TB usable capacity after RAID 5 protection|
| Network interfaces                                      | 2 X 1 GbE interface - MGMT, DATA 3. <br> MGMT - for management, not user configurable, used for initial setup <br> DATA3 - for data, user configurable, and is dynamic by default <br> MGMT and DATA 3 can also work as 10 GbE <br> 2 X 10 GbE interface - DATA 1, DATA 2 <br> Both are for data, can be configured as dynamic (default) or static |
| Data transfer media                                     | RJ45, SFP+ copper 10 GbE Ethernet  |
| Security                                                | Rugged device casing with tamper-proof custom screws <br> Tamper-evident stickers placed at the bottom of the device|
| Data transfer rate                                      | Up to 80 TB in a day over 10 GbE network interface        |
| Management                                              | Local web UI - one-time initial setup and configuration <br> Azure portal - day-to-day device management        |-->

## <a name="components"></a>元件

Data Box Heavy 包括下列元件：

* **Data Box Heavy 裝置** - 具有堅固外殼並能安全儲存資料的實體裝置。 此裝置的可用儲存體容量為 800 TB。 

    
* **資料箱服務** - Azure 入口網站的延伸模組，可讓您從不同的地理位置存取 Web 介面來管理 Data Box Heavy 裝置。 使用資料箱服務來執行 Data Box Heavy 裝置的每日管理。 服務工作包括如何建立及管理訂單、檢視及管理警示，以及管理共用。  

* **本機 Web 使用者介面** - 此 Web 型 UI 可用來設定裝置以使其可以連線到區域網路，然後再向資料箱服務註冊該裝置。 您也可以使用本機 Web UI 來將裝置關機及重新啟動、檢視複製記錄，以及連絡 Microsoft 支援服務以提出服務要求。


## <a name="the-workflow"></a>工作流程

典型的流程包含下列步驟：

1. **訂購** - 在 Azure 入口網站中建立訂單，提供運送資訊，以及資料的目的地 Azure 儲存體帳戶。 如果裝置可供使用，Azure 便會準備並寄出裝置，並附上寄送追蹤識別碼。

2. **接收** - 在裝置成功遞送之後，請使用指定的纜線將裝置接上網路及電源。 開啟電源並連線到裝置。 設定裝置網路，並在您想要複製資料的主機電腦上掛接共用。

3. **複製資料** - 將資料複製到 Data Box Heavy 共用。

4. **寄回** - 準備好裝置並將它關機，然後將裝置寄回 Azure 資料中心。

5. **上傳** - 資料會自動從裝置複製到 Azure。 裝置磁碟會根據美國國家標準與技術局 (National Institute of Standards and Technology，NIST) 的指導方針安全地抹除。

在這整個程序中，您將會透過電子郵件獲知所有狀態變更。 

## <a name="region-availability"></a>區域可用性

Data Box Heavy 可以根據其服務所部署的區域、裝置所寄送的國家/地區，以及您將資料傳輸至其中的目標 Azure 儲存體帳戶來傳輸資料。 

- **服務可用性** - 針對此版本，下列區域提供 Data Box Heavy：
    - 位於美國的所有公用雲端區域 - 美國中西部、美國西部 2、美國西部、美國中南部、美國中部、美國中北部、美國東部，以及美國東部 2。
    - 歐盟 - 西歐與北歐。
    - 英國 - 英國南部與英國西部。
    - 法國 - 法國中部與法國南部。

- **目的地儲存體帳戶** - 儲存資料的儲存體帳戶在服務可用的所有 Azure 區域都可用。 

## <a name="sign-up"></a>註冊

Data Box Heavy 處於預覽狀態，而且您必須註冊。 請執行下列步驟以註冊 Data Box Heavy︰

1. 登入 Azure 入口網站： https://aka.ms/azuredatabox。
2. 按一下 [+] 以建立新資源。 搜尋 **Azure 資料箱**。 選取 [Azure 資料箱] 服務。

    <!--![The Data Box Heavy sign up 1]()-->

3. 按一下 [建立]。

    <!--![The Data Box Heavy sign up 2]()-->

4. 挑選您要用於 Data Box Heavy 預覽的訂用帳戶。 選取您要部署 Data Box Heavy 資源的區域。 在 [Data Box Heavy] 選項中，按一下 [註冊]。

   <!--![The Data Box Heavy sign up 3]()-->

5. 回答關於資料所在國家/地區、時間範圍、資料傳輸的目標 Azure 服務、網路頻寬，以及資料傳輸頻率的問題。 檢閱 [隱私權與條款]，然後選取 [Microsoft 可以使用您的電子郵件地址連絡您] 核取方塊。

    <!--![The Data Box Heavy sign up 4]()-->

在註冊並啟用預覽後，您便可以訂購 Data Box Heavy。




