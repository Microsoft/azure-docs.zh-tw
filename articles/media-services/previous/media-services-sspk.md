---
title: Licensing Microsoft® Smooth Streaming Client Porting Kit
description: 了解如何授權 Microsoft® Smooth Streaming Client Porting Kit。
services: media-services
documentationcenter: ''
author: xpouyat
manager: cfowler
editor: ''
ms.assetid: e3b488e7-8428-4c10-a072-eb3af46c82ad
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/09/2017
ms.author: xpouyat
ms.openlocfilehash: b4472f522571e0056ce6b28d67a69b0dcabba8a6
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785817"
---
# <a name="licensing-microsoft-smooth-streaming-client-porting-kit"></a>Licensing Microsoft® Smooth Streaming Client Porting Kit
## <a name="overview"></a>概觀
Microsoft Smooth Streaming Client Porting Kit (簡稱**SSPK** ) 是最佳化的 Smooth Streaming 用戶端實作，可協助內嵌裝置製造商、有線電視和行動業者、內容服務提供者、手持式裝置製造商、獨立軟體廠商 (ISV) 和解決方案提供者打造產品和服務，以供串流 Smooth Streaming 格式的彈性內容。 SSPK 是 Smooth Streaming 用戶端的裝置和平台獨立實作，可由被授權者移植到任何裝置和平台。 

以下是一個高層級架構，而 IIS Smooth Streaming Porting Kit 方塊是 Microsoft 所提供的 Smooth Streaming 用戶端實作並包含播放 Smooth Streaming 內容的所有核心邏輯。 然而，特定裝置或平台的合作夥伴可藉由實作適當的介面來移植此內容。 

![SSPK](./media/media-services-sspk/sspk-arch.png)

## <a name="description"></a>說明
SSPK 是根據可提供絕佳商業價值的條款授權。 SSPK 授權提供給業界：

* Smooth Streaming Porting Kit 的 C++ 原始碼 
  * 實作 Smooth Streaming 用戶端功能
  * 新增格式剖析、啟發學習法、緩衝處理邏輯等等
* 播放器應用程式 API 
  * 可與媒體播放器應用程式互動的程式設計介面
* 平台抽象層 (PAL) 介面 
  * 用於與作業系統 (執行緒、通訊端) 互動的程式設計介面
* 硬體抽象層 (HAL) 介面 
  * 用於與硬體 A/V 解碼器 (解碼、轉譯) 互動的程式設計介面
* 數位版權管理 (DRM) 介面 
  * 可透過 DRM 抽象層 (DAL) 處理 DRM 的程式設計介面
  * Microsoft PlayReady Porting Kit 個別出貨，但可透過這個介面整合。 如需 Microsoft PlayReady Device 授權的詳細資訊，請按一下 [這裡](http://www.microsoft.com/playready/licensing/device_technology.mspx#pddipdl)。
* 實作範例 
  * 適用於 Linux 的 PAL 實作範例
  * 適用於 GStreamer 的 HAL 實作範例

## <a name="licensing-options"></a>授權選項
Microsoft Smooth Streaming Client Porting Kit 乃根據兩份不同的授權合約提供給被授權者：一份適用於開發 Smooth Streaming 用戶端中期產品，另一份適用於將 Smooth Streaming 用戶端最終產品散發給使用者。

* 對於需要原始程式碼移植套件來開發中期產品的晶片組製造商、系統整合業者或獨立軟體廠商 (ISV)，應該執行 Microsoft Smooth Streaming Client Porting Kit **中期產品授權** 。
* 對於需要對使用者散發 Smooth Streaming 用戶端最終產品之權利的裝置製造商或 ISV，應該執行 Microsoft Smooth Streaming Client Porting Kit **最終產品授權** 。

### <a name="microsoft-smooth-streaming-client-porting-kit-interim-product-license"></a>Microsoft Smooth Streaming Client Porting Kit 中期產品授權
Microsoft 依據本授權提供 Smooth Streaming Client Porting Kit 和必要的智慧財產權，以便開發 Smooth Streaming 用戶端中期產品並散發給其他可散發 Smooth Streaming 用戶端最終產品的 Smooth Streaming Client Porting Kit 裝置被授權者。

#### <a name="fee-structure"></a>免費結構
$50,000 美元的一次性授權費可供存取 Smooth Streaming Client Porting Kit。 

### <a name="microsoft-smooth-streaming-client-porting-kit-final-product-license"></a>Microsoft Smooth Streaming Client Porting Kit 最終產品授權
Microsoft 依據本授權提供所有必要的智慧財產權，以便從其他 Smooth Streaming Client Porting Kit 被授權者接收 Smooth Streaming 用戶端中期產品，並將公司自有品牌的 Smooth Streaming 用戶端最終產品散發給一般使用者。

#### <a name="fee-structure"></a>免費結構
Smooth Streaming 用戶端最終產品乃根據權利金模型提供，細節如下：

* 送交的每一裝置實作為 $0.10 美元
* 每年的權利金上限為 $50,000 美元
* 每年前 10,000 個裝置實作不需權利金 

## <a name="licensing-procedure-and-sspk-access"></a>授權程序和 SSPK 存取
有關所有授權查詢，請寄送電子郵件至 [sspkinfo@microsoft.com](mailto:sspkinfo@microsoft.com)。

已註冊的中期被授權者可以存取 [SSPK 發佈入口網站](https://microsoft.sharepoint.com/teams/SSPKDOWNLOAD/) 。

中期和最終 SSPK 被授權者都可以將技術問題提交到 [smoothpk@microsoft.com](mailto:smoothpk@microsoft.com)。

## <a name="microsoft-smooth-streaming-client-interim-product-agreement-licensees"></a>Microsoft Smooth Streaming 用戶端中期產品合約被授權者
* Adroit Business Solutions, Inc
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* Albis Technologies Ltd.
* Alticast Corporation
* Amazon Digital Services, Inc.
* Arion Technology, Inc.
* AVC Multimedia Software Co., Ltd.
* Cavium, Inc.
* EchoStar Purchasing Corporation
* Enseo, Inc.
* Fluendo S.A.
* HANDAN BroadInfoCom Co., Ltd.
* Infomir GMBH
* Irdeto USA Inc.
* iWEDIA S.A. 
* Liberty Global Services BV
* MediaTek Inc.
* MStar Co, Ltd
* Nintendo Co., Ltd.
* OpenTV, Inc.
* Saffron Digital Limited
* Sichuan Changhong Electric Co., Ltd
* SoftAtHome
* 索尼公司
* Tatung Technology Inc.
* TCL Technology Electronics (Huizhou) Co., Ltd.
* Top Victory Investments, Ltd.
* Vestel Elektronik Sanayi ve Ticaret A.S.
* VisualOn, Inc.
* ZTE Corporation

## <a name="microsoft-smooth-streaming-client-final-product-agreement-licensees"></a>Microsoft Smooth Streaming 用戶端最終產品合約被授權者
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* Albis Technologies Ltd.
* Amazon Digital Services, Inc.
* AmTRAN Technology Co., Ltd.
* Arcadyan Technology Corporation
* Arion Technology, Inc.
* ATMACA ELEKTRONİK SAN. VE TİC. A.Ş
* British Sky Broadcasting Limited
* CastPal Technology Inc., Shenzhen
* Compal Electronics, Inc.
* Dongguan Digital AV Technology Corp., Ltd.
* EchoStar Purchasing Corporation
* Enseo, Inc.
* Filmflex Movies Limited
* Fluendo S.A.
* Gibson Innovations Limited
* Haier Information Applicantion S.R.L
* HANDAN BroadInfoCom Co., Ltd.
* Hisense International Co., Ltd. 
* Homecast Co., Ltd
* Hon Hai Precision Industry Co., Ltd.
* Infomir GMBH
* Kaonmedia Co., Ltd.
* KDDI Corporation
* Nintendo Co., Ltd.
* Orange SA
* Saffron Digital Limited
* Sagemcom Broadband SAS
* Shenzhen Coship Electronics CO., LTD
* Shenzhen Jiuzhou Electric Co., Ltd
* Shenzhen Skyworth Digital Technology Co., Ltd
* Sichuan Changhong Electric Co., Ltd.
* 致振企業股份有限公司
* Sky Deutschland Fernsehen GmbH & Co. KG
* SmarDTV S.A.
* SoftAtHome
* 索尼公司
* TCL Overseas Marketing (Macao Commercial Offshore) Limited
* Technicolor Delivery Technologies, SAS
* Tongfang Global Ltd.
* Top Victory Investments, Ltd.
* Toshiba Lifestyle Products & Services Corporation
* Universal Media Corporation /Slovakia/ s.r.o.
* VIZIO, Inc.
* Wistron Corporation
* ZTE Corporation


## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

