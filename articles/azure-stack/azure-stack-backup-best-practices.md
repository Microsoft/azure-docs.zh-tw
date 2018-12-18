---
title: Azure Stack 基礎結構備份服務的最佳做法 | Microsoft Docs
description: 您在資料中心內部署和管理 Azure Stack 時，可以遵照一套最佳做法，以利發生重大失敗時減少資料遺失。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: 221FDE40-3EF8-4F54-A075-0C4D66EECE1A
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/01/2018
ms.author: jeffgilb
ms.reviewer: hectorl
ms.openlocfilehash: 08d8822410545fb0ae3a2a99de00b38566c9834c
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "41946512"
---
# <a name="infrastructure-backup-service-best-practices"></a>基礎結構備份服務的最佳做法

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您在資料中心內部署和管理 Azure Stack 時，可以遵照最佳做法，以利發生重大失敗時減少資料遺失。

您應該定期回顧最佳做法，以確認對作業流程進行變更後，您的安裝仍然遵照最佳做法。 若在實作這些最佳做法時遇到任何問題，請連絡 Microsoft 支援服務以取得協助。

## <a name="configuration-best-practices"></a>組態的最佳作法

### <a name="deployment"></a>部署

在部署每個 Azure Stack 雲端後，啟用基礎結構備份。 您可以使用 Azure Stack PowerShell，從任何可存取操作員管理 API 端點的用戶端/伺服器來排程備份。

### <a name="networking"></a>網路功能

路徑的通用命名慣例 (UNC) 字串必須使用完整網域名稱 (FQDN)。 若無法進行名稱解析，也可以使用 IP 位址。 UNC 字串會指定資源的位置，例如共用的檔案或裝置。

### <a name="encryption"></a>加密

加密金鑰用於將匯出到外部儲存位置的備份資料加密。 金鑰會在[使用 PowerShell 為 Azure Stack 啟用備份](azure-stack-backup-enable-backup-powershell.md)的過程中產生。

金鑰必須儲存在安全的位置 (例如，公用 Azure Key Vault 祕密中)。 在重新部署 Azure Stack 期間，必須使用此金鑰。 

![請將金鑰儲存在安全的位置。](media\azure-stack-backup\azure-stack-backup-encryption2.png)

## <a name="operational-best-practices"></a>作業的最佳作法

### <a name="backups"></a>備份

 - 基礎結構備份控制器是依需求觸發。 建議每日至少備份兩次。
 - 備份作業會在系統正在執行時進行，因此對管理體驗或使用者應用程式來說，不會有停機時間。 對於合理負載下的解決方案，備份作業預計會花費 20-40 分鐘。
 - 請使用 OEM 所提供的指示，手動備份網路交換器，此外硬體生命週期主機 (HLH) 應該儲存在與基礎結構備份控制器用以儲存控制平面備份資料相同的備份共用上。 請考慮將交換器和 HLH 設定儲存在區域資料夾中。 若您在相同區域中有多個 Azure Stack 執行個體，請考慮針對屬於同一個縮放單位的每組設定使用一組識別碼。

### <a name="folder-names"></a>資料夾名稱

 - 基礎結構會自動建立 MASBACKUP 資料夾。 這是 Microsoft 管理的共用。 您可以在與 MASBACKUP 相同的層級建立共用。 不建議您在 MASBACKUP 資料夾內，建立 Azure Stack 未建立的資料夾或儲存體資料。 
 -  請使用資料夾名稱中的 FQDN 和區域，區分來自不同雲端的備份資料。 您 Azure Stack 部署和端點的完整網域名稱 (FQDN) 是「區域」參數和「外部網域名稱」參數的組合。 如需詳細資訊，請參閱 [Azure Stack 資料中心整合 - DNS](azure-stack-integrate-dns.md)。

例如，備份共用是裝載在 fileserver01.contoso.com 上的 AzSBackups。 在該檔案共用中，每個 Azure Stack 部署可能都會有一個使用外部網域名稱的資料夾，和一個使用區域名稱的子資料。 

FQDN：contoso.com  
區域：nyc


    \\fileserver01.contoso.com\AzSBackups
    \\fileserver01.contoso.com\AzSBackups\contoso.com
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\MASBackup

MASBackup 資料夾是 Azure Stack 儲存其備份資料的地方。 請勿使用此資料夾來儲存您自己的資料。 OEM 也不應該使用此資料夾來儲存任何備份資料。 

我們鼓勵 OEM 將其元件的備份資料，儲存在區域資料夾底下。 每個網路交換器、硬體生命週期主機 (HLH) 等等，都可以儲存在其專屬的子資料夾中。 例如︰

    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\HLH
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\Switches
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\DeploymentData
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\Registration

### <a name="monitoring"></a>監視

系統支援下列警示：

| 警示                                                   | 說明                                                                                     | 補救                                                                                                                                |
|---------------------------------------------------------|-------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| 由於檔案共用容量不足，因此備份失敗 | 由於檔案共用容量不足，因此備份控制器無法將備份檔案匯出到該位置。 | 請新增更多的儲存容量，然後重試備份。 請刪除現有的備份 (從最舊的備份開始) 以釋出空間。                    |
| 由於連線問題，因此備份失敗。             | Azure Stack 與檔案共用之間的網路發生問題。                          | 請解決網路問題，然後重試備份。                                                                                            |
| 由於路徑有誤，因此備份失敗                | 無法解析檔案共用路徑                                                          | 請從不同的電腦對應共用，以確保共用是可存取的。 若路徑已不再有效，您可能會需要更新路徑。       |
| 由於驗證問題，因此備份失敗               | 認證可能有問題，或有網路問題影響驗證。    | 請從不同的電腦對應共用，以確保共用是可存取的。 若認證已不再有效，您可能會需要更新認證。 |
| 由於一般錯誤，因此備份失敗                    | 可能是因為間歇性問題而導致要求失敗。 請重試備份。                    | 請致電支援部門                                                                                                                               |

## <a name="next-steps"></a>後續步驟

 - 請檢閱[基礎結構備份服務](azure-stack-backup-reference.md)的參考資料。  
 - 啟用[基礎結構備份服務](azure-stack-backup-enable-backup-console.md)。
