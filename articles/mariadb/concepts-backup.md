---
title: 在適用於 MariaDB 的 Azure 資料庫中備份與還原
description: 了解自動備份及還原您適用於 MariaDB 的 Azure 資料庫伺服器。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: c8e671071226b891668e0874d56e2d4685e1d64d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958575"
---
# <a name="backup-and-restore-in-azure-database-for-mariadb"></a>在適用於 MariaDB 的 Azure 資料庫中備份與還原

適用於 MariaDB 的 Azure 資料庫會自動建立伺服器備份，並將其儲存在使用者設定的本地備援或異地備援儲存體中。 備份可以用來將伺服器還原至某個時間點。 備份和還原可保護資料免於意外損毀或刪除，是商務持續性策略中不可或缺的一部分。

## <a name="backups"></a>備份

適用於 MariaDB 的 Azure 資料庫採用完整、差異及交易記錄備份。 在您設定的備份保留期限內，這些備份可讓您將伺服器還原至任何時間點。 預設的備份保留期限是七天。 可選擇設定的期限最多為 35 天。 所有備份皆會使用 AES 256 位元加密進行加密。

### <a name="backup-frequency"></a>備份頻率

一般而言，完整備份每週執行一次、差異備份每天執行兩次，而記錄備份每五分鐘執行一次。 建立伺服器之後，會立即排程第一次完整備份。 在還原大量資料的伺服器上，初次備份可能需要較長時間。 可在其中還原新伺服器的最早時間點，是完成初次完整備份的時間。

### <a name="backup-redundancy-options"></a>備份備援選項

適用於 MariaDB 的 Azure 資料庫可讓您在一般用途和記憶體最佳化層中，彈性地選擇本地備援或異地備援備份儲存體。 當備份儲存在異地備援備份儲存體中時，這些備份不會只儲存在裝載您伺服器的區域內，也會複寫至[配對的資料中心](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。 這樣能提供更好的保護性和功能，當發生災害時，您就可以在不同區域中還原伺服器。 「基本」層只會提供本地備援的備份儲存體。

> [!IMPORTANT]
> 您只可在伺服器建立期間，為備份設定本地備援或異地備援儲存體。 伺服器佈建完成之後，您無法變更備份儲存體備援選項。

### <a name="backup-storage-cost"></a>備份儲存體成本

適用於 MariaDB 的 Azure 資料庫可提供高達 100% 的已佈建伺服器儲存體作為備份儲存體，且不須支付額外費用。 一般而言，這適用於 7 天備份保留期。 使用任何其他備份儲存體會以「GB/月」計費。

例如，如果您已佈建的伺服器大小為 250 GB，您就能免費獲得 250 GB 的備份儲存體。 超過 250 GB 的儲存體則會收費。

如需備份儲存體成本的詳細資訊，請前往 [MariaDB 定價頁面](https://azure.microsoft.com/pricing/details/mariadb/)。

## <a name="restore"></a>Restore

在適用於 MariaDB 的 Azure 資料庫中，執行還原會從原始伺服器的備份中建立新伺服器。

有兩種類型的還原可使用：

- **時間點還原**可搭配任一備份備援選項使用，並在您原始伺服器所在區域中建立新的伺服器。
- **異地還原**只能您將伺服器設定為使用異地備援儲存體時使用，這可讓您將伺服器還原至不同的區域。

預估的復原時間取決於數個因素，包括資料庫大小、交易記錄大小、網路頻寬，以及在相同區域中同時進行復原的資料庫總數。 復原時間通常不到 12 小時。

> [!IMPORTANT]
> 已刪除的伺服器**無法**還原。 如果您刪除伺服器，所有屬於該伺服器的資料庫也會一併刪除，且無法復原。

### <a name="point-in-time-restore"></a>還原時間點

與備份備援選項無關，您可以在備份保留期限內地任何時間點執行還原。 新伺服器會建立在與原始伺服器相同的 Azure 區域中。 其使用原始伺服器的組態來建立，包含定價層、計算世代、虛擬核心數目、儲存體大小、備份保留期限，以及備份備援選項。

時間點還原適用於多種案例。 例如，使用者不小心刪除資料、卸除重要的資料表或資料庫，或是因為應用程式缺失，使應用程式不小心用不正確的資料覆寫正確資料。

您可能需要等候下一個交易記錄備份執行時，才能還原至前五分鐘內的時間點。

### <a name="geo-restore"></a>異地還原

如果您已將伺服器設定為使用異地備援備份，您可以將伺服器還原到另一個可使用服務的 Azure 區域中。 當您的伺服器因為裝載伺服器區域中的事件而無法使用時，異地還原就是預設的復原選項。 如果區域中的大規模意外導致您無法使用資料庫應用程式，則您可以從異地備援備份，將伺服器還原到任何其他區域中的伺服器。 在建立備份及將它複寫至不同區域之間會有延遲。 此延遲可能最長達一小時，因此當發生災害時，最多可能會遺失最長達一小時的資料。

在異地還原期間，可以進行變更的伺服器設定包括計算世代、vCore、備份保留期間及備份備援選項。 異地還原期間不支援變更定價層 (基本、一般，或記憶體最佳化) 或儲存體大小。

### <a name="perform-post-restore-tasks"></a>執行還原之後的工作

從其中任何一種復原機制還原之後，您應執行下列工作，讓您的使用者和應用程式回復正常執行狀態︰

- 如果新伺服器就會取代原始伺服器，則將用戶端和用戶端應用程式重新導向至新伺服器
- 確定有適當的伺服器層級防火牆規則供使用者連線
- 確定有適當的登入和資料庫層級權限
- 依適當情況設定警示

## <a name="next-steps"></a>後續步驟

- 若要深入了解商務持續性，請參閱 [商務持續性概觀](concepts-business-continuity.md)。
- 若要使用 Azure 入口網站還原至某個時間點，請參閱 [使用 Azure 入口網站將資料庫還原至時間點](howto-restore-server-portal.md)。
 
<!--
- To restore to a point in time using Azure CLI, see [restore database to a point in time using CLI](howto-restore-server-cli.md).-->