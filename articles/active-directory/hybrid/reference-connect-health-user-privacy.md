---
title: Azure AD Connect Health 與使用者隱私權 | Microsoft Docs
description: 本文件將說明使用者隱私權與 Azure AD Connect Health。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/26/2018
ms.author: billmath
ms.openlocfilehash: 7d3f625524161703b7be822c1db0fd5b21030dca
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306347"
---
# <a name="user-privacy-and-azure-ad-connect-health"></a>使用者隱私權與 Azure AD Connect Health 

[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

>[!NOTE] 
>本文會討論 Azure AD Connect Health 與使用者隱私權。  如需 Azure AD Connect 與使用者隱私權的相關資訊，請參閱[這裡](reference-connect-user-privacy.md)的文章。

## <a name="user-privacy-classification"></a>使用者隱私權分類
Azure AD Connect Health 會歸類到 GDPR 分類的**資料處理器**類別。 作為資料處理器管線，服務會對重要合作夥伴和終端取用者提供資料處理服務。 Azure AD Connect Health 不會產生使用者資料，也不會自行控制要收集哪些個人資料以及如何使用這些資料。 在 Azure AD Connect Health 中，資料的擷取、彙總、分析和報告都會以現有內部部署資料來作為基礎。 

## <a name="data-retention-policy"></a>資料保留原則
Azure AD Connect Health 不會針對超過 30 天的資料產生報告、執行分析或提供見解。 因此，Azure AD Connect Health 不會儲存、處理或保留任何資料達 30 天以上。 這項設計符合 GDPR 法規、Microsoft 隱私權合規性規定和 Azure AD 資料保留原則。 

伺服器如果連續 30 天有作用中「健全狀況服務的資料並非最新」**錯誤**警示，即表示在該時間範圍內，沒有任何資料抵達 Connect Health。 這些伺服器將會被停用，而不會顯示在 Connect Health 入口網站中。 若要重新啟用這些伺服器，您必須[將健康情況代理程式解除安裝後再重新安裝](how-to-connect-health-agent-install.md)。 請注意，這並不適用於具有相同警示類型的**警告**。 警告代表的是您收到警示的伺服器中遺失部分資料。 
 
## <a name="disable-data-collection-and-monitoring-in-azure-ad-connect-health"></a>在 Azure AD Connect Health 中停用資料收集和監視
Azure AD Connect Health 可讓您停止針對每個個別監視的伺服器或針對一個所監視服務的執行個體收集資料。 例如，您可以針對使用 Azure AD Connect Health 所監視的個別 ADFS (Active Directory 同盟服務) 伺服器停止資料收集。 您也可以針對使用 Azure AD Connect Health 所監視的整個 ADFS 執行個體停止資料收集。 當您選擇這樣做時，系統就會在停止資料收集後從 Azure AD Connect Health 入口網站中刪除對應的伺服器。 

>[!IMPORTANT]
> 您需要 Azure AD 全域系統管理員權限或 RBAC 的參與者角色，才能從 Azure AD Connect Health 刪除所監視的伺服器。
>
> 從 Azure AD Connect Health 移除服務或服務執行個體的動作無法復原。 

### <a name="what-to-expect"></a>有何影響？
如果您要針對個別監視的伺服器或一個所監視服務的執行個體停止資料收集和監視，請注意下列事項：

- 當您刪除所監視服務的執行個體時，系統會從入口網站的 Azure AD Connect Health 監視服務清單中移除該執行個體。 
- 當您刪除所監視的伺服器或所監視服務的執行個體時，Health 代理程式「不會」解除安裝，也「不會」從您的伺服器中移除。 Health 代理程式的設定是不要將資料傳送至 Azure AD Connect Health。 您必須在先前所監視的伺服器上手動解除安裝 Health 代理程式。
- 如果您在執行此步驟之前還未解除安裝 Health 代理程式，您可能會在伺服器上看到與 Health 代理程式相關的錯誤事件。
- 根據 Microsoft Azure 資料保留原則，系統會刪除屬於所監視服務執行個體的所有資料。

### <a name="disable-data-collection-and-monitoring-for-an-instance-of-a-monitored-service"></a>對所監視服務的執行個體停用資料收集和監視
請參閱[如何從 Azure AD Connect Health 刪除服務執行個體](how-to-connect-health-operations.md#delete-a-service-instance-from-azure-ad-connect-health-service)。

### <a name="disable-data-collection-and-monitoring-for-a-monitored-server"></a>對所監視的伺服器停用資料收集和監視
請參閱[如何從 Azure AD Connect Health 移除伺服器](how-to-connect-health-operations.md#delete-a-server-from-the-azure-ad-connect-health-service)。

### <a name="disable-data-collection-and-monitoring-for-all-monitored-services-in-azure-ad-connect-health"></a>在 Azure AD Connect Health 中停用所有受監視服務的資料收集和監視
Azure AD Connect Health 也提供選項來停止租用戶中**所有**已註冊服務的資料收集。 我們建議您仔細考慮，並且在所有全域管理員完全認可後，再採取動作。 程序一旦開始，Connect Health 服務會停止接收、處理和報告所有服務的任何資料。 Connect Health 服務中的現有資料不能保留超過 30 天。
如果您想要停止特定伺服器的資料收集，請依照刪除特定伺服器上的步驟。 若要停止租用戶方面的資料收集，請遵循下列步驟來停止資料收集和刪除租用戶的所有服務。

1.  在主要刀鋒視窗的組態下，按一下 [一般設定]。 
2.  在刀鋒視窗頂端，按一下 [停止資料收集] 按鈕。 在程序啟動之後，租用戶組態設定的其他選項將會停用。  
 
 ![停止資料收集](./media/reference-connect-health-user-privacy/gdpr4.png)
  
3.  確認會受到停止資料收集影響的上架服務清單。 
4.  輸入確切的租用戶名稱，以啟用 [刪除] 動作按鈕
5.  按一下 [刪除] 以觸發所有服務的刪除作業。 Connect Health 將會停止接收、處理、報告從已上架服務傳出的任何資料。 整個程序可能需要多達 24 小時的時間。 請注意，此步驟無法復原。 
6.  此程序完成之後，您將不會在 Connect Health 中看到任何已註冊的服務。 

 ![資料收集停止之後](./media/reference-connect-health-user-privacy/gdpr5.png)

## <a name="re-enable-data-collection-and-monitoring-in-azure-ad-connect-health"></a>在 Azure AD Connect Health 中重新啟用資料收集和監視
若要在 Azure AD Connect Health 中對先前刪除的所監視服務重新啟用監視，您必須在所有伺服器上先解除安裝再[重新安裝 Health 代理程式](how-to-connect-health-agent-install.md)。

### <a name="re-enable-data-collection-and-monitoring-for-all-monitored-services"></a>對所監視的所有服務重新啟用資料收集和監視

您可以在 Azure AD Connect Health 中恢復租用戶方面的資料收集。 我們建議您仔細考慮，並且在所有全域管理員完全認可後，再採取動作。

>[!IMPORTANT]
> 下列步驟必須在執行停用後的 24 小時以後才能進行。
> 啟用資料收集之後，Connect Health 中顯示的見解和監視資料將不包括以前收集的舊資料。 

1.  在主要刀鋒視窗的組態下，按一下 [一般設定]。 
2.  在刀鋒視窗頂端，按一下 [啟用資料收集] 按鈕。 
 
 ![啟用資料收集](./media/reference-connect-health-user-privacy/gdpr6.png)
 
3.  輸入確切的租用戶名稱，以啟動 [啟用] 按鈕。
4.  按一下 [啟用] 按鈕，以授與 Connect Health 服務中的資料收集權限。 變更會在短時間內套用。 
5.  請遵循[安裝程序](how-to-connect-health-agent-install.md)，在伺服器中重新安裝要監視的代理程式，服務將會出現在入口網站中。  


## <a name="next-steps"></a>後續步驟
* [在信任中心檢閱 Microsoft 隱私權原則](https://www.microsoft.com/trustcenter)
* [Azure AD Connect 與使用者隱私權](reference-connect-user-privacy.md)

