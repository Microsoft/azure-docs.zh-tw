---
title: 評估從 StorSimple 5000-7000 系列移轉資料的選項 | Microsoft Docs
description: 概略說明從 StorSimple 5000-7000 系列移轉資料的選項。
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/23/2018
ms.author: alkohli
ms.openlocfilehash: 8f34d5a38f09f015547f52cc4b44819b780932bb
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818853"
---
# <a name="options-to-migrate-data-from-storsimple-5000-7000-series"></a>從 StorSimple 5000-7000 系列移轉資料的選項 

> [!IMPORTANT]
> StorSimple 5000/7000 系列將於 2019 年 7 月 31 日到達結束支援 (EOS) 狀態。 建議 StorSimple 5000/7000 系列的客戶遷移至文件中所述的其中一個替代方案。

StorSimple 5000-7000 系列即將在 2019 年 7 月[終止支援](https://support.microsoft.com/lifecycle/search?alpha=StorSimple%205000%2F7000%20Series)。 執行 StorSimple 5000-7000 系列的客戶可以選擇升級至其他 Azure 第一方混合式服務。 本文說明可用來移轉資料的 Azure 混合式選項。 

## <a name="migration-options"></a>移轉選項

使用 StorSimple 5000-7000 系列的客戶有下列兩個主要選項：

- **升級至 StorSimple 8000 系列** – 升級至 StorSimple 8000 系列，並繼續使用 StorSimple 平台。  客戶若選擇此升級路徑，即必須將其 5000-7000 系列裝置取代為 8000 系列。 資料可透過移轉工具從 5000-7000 系列裝置進行移轉。 移轉順利完成後，StorSimple 8000 系列裝置會繼續將資料分層至 Azure Blob 儲存體。 

    如需如何使用 StorSimple 8000 系列移轉資料的詳細資訊，請移至[將資料從 StorSimple 5000-7000 系列移轉至 8000 系列裝置](storsimple-8000-migrate-from-5000-7000.md)。

- **移轉至 Azure 檔案同步** – 這個全新的移轉選項可讓客戶將其組織的檔案共用儲存在 Azure 檔案服務中。 這些檔案共用隨後可集中管理，以便使用 Azure 檔案同步 (AFS) 進行內部部署存取。 AFS 可部署在 Windows Server 主機上。 其後，可以透過主機複本或使用移轉工具來執行實際資料移轉。

    如需如何將資料移轉至 Azure 檔案同步的詳細資訊，請移至[將資料從 StorSimple 5000-7000 系列移轉至 Azure 檔案同步](https://aka.ms/StorSimpleMigrationAFS)。

## <a name="migration---frequently-asked-questions"></a>移轉 - 常見問題集

### <a name="q-when-do-the-storsimple-5000-and-7000-series-devices-reach-end-of-service"></a>問： StorSimple 5000 和 7000 系列裝置的服務將於何時終止？ 

A. StorSimple 5000-7000 系列將在 2019 年 7 月[終止服務](https://support.microsoft.com/lifecycle/search?alpha=StorSimple%205000%2F7000%20Series)。 終止服務意味著在 2019 年 7 月之後，Microsoft 將無法再提供這些裝置的軟硬體支援。 強烈建議您立即開始擬定從裝置移轉資料的計劃。

### <a name="q-what-happens-to-the-data-i-have-stored-in-azure"></a>問： 我已儲存在 Azure 中的資料將會如何？  

A. 將資料移轉至較新的服務後，您即可繼續在 Azure 中使用這些資料。 


### <a name="q--what-happens-to-the-data-i-have-stored-locally-on-my-storsimple-device"></a>問：  我已在 StorSimple 裝置的本機儲存的資料將會如何？ 

A. 本機裝置上的資料可複製到較新的服務，如移轉文件所說明。

### <a name="what-happens-if-i-want-to-keep-my-storsimple-50007000-series-appliance"></a>如果我想要保留 StorSimple 5000/7000 系列設備，會有何後果？ 

A. 雖然服務或許可繼續運作，但 Microsoft 將無法再提供軟硬體支援。 為保有商務持續性，強烈建議您進行移轉。

### <a name="q-what-options-are-available-to-migrate-data-from-storsimple-5000-7000-series-devices"></a>問： 從 StorSimple 5000-7000 系列 裝置移轉資料的選項有哪些？ 

A. StorSimple 5000-7000 系列使用者可根據其所處的情境，選擇下列移轉選項。 

 - **升級至 8000 系列**：如果您想要繼續使用 StorSimple 平台，請使用此選項。 
 - **移轉至 Azure 檔案同步**：如果您想要切換至 Azure 原生格式，請使用此選項。 您可以使用 Azure 檔案同步來集中管理檔案共用。 

您可以連絡 Microsoft 支援服務，討論此處未列出的移轉選項。

### <a name="q-is-migration-to-other-storage-solutions-supported"></a>問： 是否支援移轉至其他儲存體解決方案？

A. 是。 使用資料的主機複本移轉至其他儲存體解決方案，是可行的。

### <a name="q-is-migration-supported-by-microsoft"></a>問： Microsoft 是否支援移轉？ 

A. 從 5000 或 7000 系列移轉的作業受到完整支援。 事實上，Microsoft 建議您在開始移轉之前，應先洽詢「支援服務」。 移轉目前是協助進行的作業。 如果您想要從 StorSimple 5000-7000 系列裝置移轉資料，請[建立支援票證](storsimple-8000-contact-microsoft-support.md)。

### <a name="q-how-does-the-cost-compare-for-the-two-listed-migrations-to-azure-hybrid-services"></a>問： 此處所列的兩種 Azure 混合式服務移轉在成本上有何差異？ 

A. 移轉的成本取決於您選擇的選項。 雖然移轉作業本身是免費的，但如果您決定要升級到 StorSimple 8000 系列，則會有硬體裝置的成本。 同樣地，在使用 Azure 檔案同步時，將會收取該服務的訂閱費用。 在前述兩種情況下，客戶也都必須支付現行儲存體的成本。 請參閱 [Microsoft 個別服務的定價計算機](https://azure.microsoft.com/pricing/#product-picker)，以進行估算。  

### <a name="q--how-long-does-it-take-to-complete-a-migration"></a>問：  完成移轉需要多少時間呢？

A. 移轉資料的所需時間，取決於資料量和選取的升級選項。 

## <a name="next-steps"></a>後續步驟
 - [將資料從 StorSimple 5000-7000 系列移轉至 8000 系列裝置](storsimple-8000-migrate-from-5000-7000.md)。
 - [將資料從 StorSimple 5000-7000 系列移轉至 Azure 檔案同步](storsimple-5000-7000-afs-migration.md)
