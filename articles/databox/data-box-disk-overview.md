---
title: Microsoft Azure 資料箱磁碟概觀 | 資料中的 Microsoft Docs
description: 說明 Azure 資料箱磁碟，這是可讓您將大量資料傳輸至 Azure 的雲端解決方案
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: overview
ms.date: 09/04/2018
ms.author: alkohli
Customer intent: As an IT admin, I need to understand what Data Box Disk is and how it works so I can use it to import on-premises data into Azure.
ms.openlocfilehash: 322c398bd6384821697e2b672825cc0805eb43ad
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092893"
---
# <a name="what-is-azure-data-box-disk-preview"></a>什麼是 Azure 資料箱磁碟？ (預覽)

Microsoft Azure 資料箱磁碟解決方案可讓您以快速、實惠和可靠的方式，將數 TB 的內部部署資料傳送至 Azure。 安全的資料傳輸可透過寄送給您的 1 到 5 個固態硬碟 (SSD) 加速運作。 這些 8 TB 的加密磁碟會透過區域貨運公司傳送至您的資料中心。 

您可以透過 Azure 入口網站中的資料箱服務快速設定、連接和解除鎖定這些磁碟。 請將資料複製到磁碟，然後將磁碟送回給 Azure。 在 Azure 資料中心內，您的資料會透過快速的私人網路上傳連結從磁碟機自動上傳至雲端。


> [!IMPORTANT]
> - 資料箱磁碟處於預覽狀態。 部署此解決方案之前，請先檢閱 [Azure 預覽版服務條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 
> - 您必須註冊此服務。 若要註冊，請移至[預覽版入口網站](http://aka.ms/azuredataboxfromdiskdocs)。
> - 在預覽期間，資料箱磁碟可運送給美國與歐盟的客戶。 如需詳細資訊，請移至[區域可用性](#region-availability)。

## <a name="use-cases"></a>使用案例

您可以在沒有網路連線或網路連線受限的情況下，使用資料箱磁碟傳輸數 TB 的資料。 資料移動可以是單次或定期的，或一開始是大量資料傳輸，然後繼之以定期傳輸。 

- **單次移轉** - 有大量的內部部署資料移至 Azure 時。 例如，將資料從離線磁帶移至 Azure 非經常性存取儲存體中的封存資料。
- **增量傳輸** - 在使用資料箱磁碟 (種子) 完成初始大量傳輸後，繼之以透過網路的增量傳輸時。 例如，使用 Commvault 和資料箱磁碟將備份複本移至 Azure。 在這項移轉之後，會透過網路將累加的資料複製到 Azure 儲存體。 
- **定期上傳** - 定期產生大量資料而需要移至 Azure 時。 例如，在進行能源探勘時，鑽井平台和風車農場會產生視訊內容。

## <a name="the-workflow"></a>工作流程

典型的流程包含下列步驟：

1. **訂購** - 在 Azure 入口網站中建立訂單，提供運送資訊，以及資料的目的地 Azure 儲存體帳戶。 如果有可用的磁碟，Azure 即會加密、準備並運送附有出貨追蹤識別碼的磁碟。

2. **接收** - 在磁碟送達後，請打開包裝，並將磁碟連線至要複製的資料存放所在的電腦。 將磁碟解除鎖定。
    
3. **複製資料** - 藉由拖放將資料複製到磁碟上。

4. **送回** - 備妥磁碟並將其送回至 Azure 資料中心。

5. **上傳** - 資料會自動從磁碟複製到 Azure。 磁碟會根據美國國家標準與技術局 (NIST) 的指導方針安全地清除。

在這整個程序中，您將會透過電子郵件獲知所有狀態變更。 如需更多有關詳細流程的資訊，請移至[在 Azure 入口網站中部署資料箱磁碟](data-box-disk-quickstart-portal.md)。


## <a name="benefits"></a>優點

資料箱磁碟依設計可將大量資料移至 Azure，而不會影響到網路。 此解決方案有下列優點：

- **速度** - 資料箱磁碟可使用 USB 3.0 連線在一週以內將高達 35 TB 的資料移至 Azure。   

- **容易使用** - 資料箱簡單易用的解決方案。

    - 磁碟會使用 USB 連線，幾乎不需要花時間設定。
    - 磁碟的板型規格精巧，可讓您輕鬆地操作。
    - 磁碟不需要外接電源。
    - 磁碟可用於資料中心伺服器、桌上型電腦或膝上型電腦。
    - 此解決方案會透過 Azure 入口網站提供端對端追蹤。    

- **安全** - 資料箱磁碟具有適用於磁碟、資料和服務的內建安全保護機制。 
    - 磁碟可防竄改，並且支援安全更新功能。 
    - 磁碟上的資料會全程受到 AES 128 位元加密的保護。 
    - 磁碟只能使用 Azure 入口網站中提供的金鑰來解除鎖定。 
    - 此服務受到 Azure 安全功能的保護。 
    - 在您的資料上傳至 Azure 後，磁碟即會根據 NIST 800-88r1 標準進行清理。  
    
如需詳細資訊，請移至 [Azure 資料箱磁碟安全性和資料保護](data-box-disk-security.md)。


## <a name="features-and-specifications"></a>功能和規格


| 規格                                          | 說明              |
|---------------------------------------------------------|--------------------------|
| Weight                                                  | < 2 磅。 每個資料箱。 資料箱中最多有 5 個磁碟                |
| 維度                                              | 磁碟 - 2.5 吋 SSD |            
| 纜線                                                  | 每個磁碟有 1 條 USB 3.1 纜線|
| 每份訂單的儲存體容量                              | 40 TB (可用容量最高 35 TB)|
| 磁碟儲存體容量                                   | 8 TB (可用容量最高 7 TB)|
| 資料介面                                          | USB   |
| 安全性                                                | 使用 BitLocker 預先加密以及安全更新 <br> 受通行金鑰保護的磁碟 <br> 資料全程加密  |
| 資料傳輸速率                                      | 最高 430 MBps，視檔案大小而定      |
|管理性                                               | Azure 入口網站 |


## <a name="region-availability"></a>區域可用性

在預覽期間，資料箱磁碟可將資料傳輸至下列 Azure 區域：


|Azure 區域  |Azure 區域  |
|---------|---------|
|美國中西部     |加拿大中部       |        
|美國西部 2     |加拿大東部         |     
|美國西部     | 西歐        |      
|美國中南部   |北歐     |         
|美國中部     |澳洲東部|
|美國中北部  |澳大利亞東南部   |
|美國東部      |澳大利亞中部 |
|美國東部 2     |澳大利亞中部 2|


## <a name="pricing"></a>價格

在預覽期間，資料箱磁碟可供免費使用。 當資料箱磁碟正式推出後，此機制將可能有所變更。

## <a name="next-steps"></a>後續步驟

- 檢閱[資料箱磁碟需求](data-box-disk-system-requirements.md)。
- 了解[資料箱磁碟限制](data-box-disk-limits.md)。
- 在 Azure 入口網站中快速部署 [Azure 資料箱磁碟](data-box-disk-quickstart-portal.md)。
