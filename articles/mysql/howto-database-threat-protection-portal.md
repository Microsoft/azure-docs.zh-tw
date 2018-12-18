---
title: 進階威脅防護 - 適用於 MySQL 的 Azure 資料庫 | Microsoft Docs
description: 「威脅保護」會偵測異常資料庫活動，指出資料庫有潛在的安全性威脅。
services: mysql
author: bolzmj
manager: kfile
ms.service: mysql
ms.topic: article
ms.date: 09/20/2018
ms.author: mbolz
ms.openlocfilehash: dfb4d104d3dfdb3e6ae7466fa3776dd5e7155aa7
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045781"
---
# <a name="advanced-threat-protection-for-azure-database-for-mysql"></a>適用於 MySQL 的 Azure SQL 資料庫進階威脅防護

適用於 MySQL 的 Azure 資料庫進階威脅防護偵測到了異常活動，這表示有不尋常及可能有害的活動試圖存取或惡意探索資料庫。

「進階威脅防護」是進階資料安全性供應項目的一部分，該供應項目是進階安全性功能的整合套件。 進階威脅防護可透過 [Azure 入口網站](https://portal.azure.com)存取和管理，而且目前為預覽狀態。

> [!NOTE]
> 下列 Azure Government 和主權雲端區域**無法**使用進階威脅防護功能：US Gov 德克薩斯州、US Gov 亞利桑那州、US Gov 愛荷華州、US Gov 維吉尼亞州、US DoD 東部、US DoD 中部、德國中部、德國北部、中國東部、中國東部 2。 如需一般產品可用性，請瀏覽[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。
>

## <a name="set-up-threat-detection"></a>設定威脅偵測
1. 在 [https://portal.azure.com](https://portal.azure.com) 上啟動 Azure 入口網站。
2. 瀏覽至所要保護的 適用於 MySQL 的 Azure 資料庫伺服器的設定頁面。 在 [安全性] 設定中，選取 [進階威脅防護 (預覽)]。
3. 在 [進階威脅防護 (預覽)] 設定頁面上：

   - 在伺服器上啟用進階威脅防護。
   - 在 [進階威脅防護設定] 的 [傳送警示給] 文字方塊中，提供要在偵測到異常資料庫活動時收到安全性警示的電子郵件清單。
  
   ![設定威脅偵測](./media/howto-database-threat-protection-portal/set-up-threat-protection.png)

## <a name="explore-anomalous-database-activities"></a>探索異常資料庫活動

偵測到異常資料庫活動時，您會收到電子郵件通知。 電子郵件會提供可疑安全性事件的相關資訊，包括異常活動的性質、資料庫名稱、伺服器名稱、應用程式名稱和事件時間。 此外，該電子郵件還會提供可能原因和建議動作的相關資訊，以協助您調查和減輕資料庫的潛在威脅。
 
1. 按一下電子郵件中的 [檢視最近的警示] 連結來啟動 Azure 入口網站，並顯示 Azure 資訊安全中心警示頁面，其中會概述在 SQL 資料庫上偵測到的作用中威脅。
    
    ![異常活動報告](./media/howto-database-threat-protection-portal/anomalous-activity-report.png)

    檢視作用中的威脅：

    ![作用中的威脅](./media/howto-database-threat-protection-portal/active-threats.png)

2. 按一下特定警示可取得其他詳細資料和調查此威脅的建議，並對未來的威脅採取補救措施。
    
    ![特定警示](./media/howto-database-threat-protection-portal/specific-alert.png)

## <a name="explore-threat-detection-alerts"></a>探索威脅偵測警示

SQL Database 威脅偵測將自有的警示與 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)整合。 在 Azure 入口網站中，資料庫和 SQL ATP 刀鋒視窗內的 SQL 動態威脅偵測圖格會追蹤作用中威脅的狀態。

按一下 [威脅偵測警示] 會啟動 Azure 資訊安全中心的警示頁面，並獲得在資料庫中偵測到的作用中 SQL 威脅概觀。

   ![威脅偵測警示](./media/howto-database-threat-protection-portal/threat-detection-alert-asc.png)
   

## <a name="next-steps"></a>後續步驟

* 深入了解 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)
* 如需定價的詳細資訊，請參閱[適用於 MySQL 的 Azure 資料庫價格頁面](https://azure.microsoft.com/pricing/details/mysql/)  
