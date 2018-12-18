---
title: 使用 Azure AD Connect Health 搭配 AD DS | Microsoft Docs
description: 這是 Azure AD Connect Health 頁面，其中討論如何監視 AD DS。
services: active-directory
documentationcenter: ''
author: zhiweiw
manager: mtillman
editor: curtand
ms.assetid: 19e3cf15-f150-46a3-a10c-2990702cd700
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/18/2017
ms.author: billmath
ms.openlocfilehash: d2ccefd1ecb83011763925abfe004d8e546f025d
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46301831"
---
# <a name="using-azure-ad-connect-health-with-ad-ds"></a>在 AD DS 使用 Azure AD Connect Health
下列文件適用於使用 Azure AD Connect Health 來監視 Active Directory 網域服務。 AD DS 支援的版本：Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2 及 Windows Server 2016。

如需使用 Azure AD Connect Health 來監視 AD FS 的詳細資訊，請參閱[在 AD FS 使用 Azure AD Connect Health](how-to-connect-health-adfs.md)。 此外，如需使用 Azure AD Connect Health 來監視 Azure AD Connect (同步處理) 的詳細資訊，請參閱 [使用適用於同步處理的 Azure AD Connect Health](how-to-connect-health-sync.md)。

![適用於 AD DS 的 Azure AD Connect Health](./media/how-to-connect-health-adds/domainservicesnapshot.PNG)

## <a name="alerts-for-azure-ad-connect-health-for-ad-ds"></a>適用於 AD DS 的 Azure AD Connect Health 警示
適用於 AD DS 的 Azure AD Connect Health 中 [警示] 區段提供的作用中和已解決警示清單，與您的網域控制站相關。 選取作用中或已解決的警示會開啟新的刀鋒視窗，內含其他資訊和解決步驟以及支援文件的連結。 每個警示類型可以有一或多個執行個體，會對應到受該特定警示影響的各個網域控制站。 在靠近警示刀鋒視窗的底部，您可以連按兩下受影響的網域控制站來開啟額外的刀鋒視窗，內含有關該警示執行個體的其他詳細資訊。

在此刀鋒視窗內，您可以啟用警示的電子郵件通知，以及變更檢視中的時間範圍。 展開時間範圍可讓您查看先前已解決的警示。

![Azure AD Connect 同步處理錯誤](./media/how-to-connect-health-adds/aadconnect-health-adds-alerts.png)

## <a name="domain-controllers-dashboard"></a>網域控制站儀表板
此儀表板提供環境的拓撲檢視，以及關鍵作業計量和每個受監視網域控制站的健康狀態。 顯示的計量有助於快速識別任何可能需要進一步調查的網域控制站。 根據預設，只會顯示部份的資料行。 不過，連按兩下資料行命令，即可找到整組可用的資料行。 選取您最想關注的資料行，即可在儀表板中單一且輕鬆地檢視 AD DS 環境的健康狀態。

![網域控制站](./media/how-to-connect-health-adds/aadconnect-health-adds-domainsandsites-dashboard.png)

網域控制站可依其各別的網域或網站分組，有助您瞭解環境拓撲。 最後，如果連按兩下刀鋒視窗標頭，儀表板便會最大化以利用可用的螢幕畫面。 顯示多個資料行時，此較大的檢視很有幫助。

## <a name="replication-status-dashboard"></a>複寫狀態儀表板
此儀表板可檢視複寫狀態和受監視網域控制站的複寫拓撲。 最新的複寫嘗試狀態會列出來，並附上針對所發現錯誤的有用文件。 您可以連按兩下包含錯誤的網域控制站，以開啟內含下列資訊的新刀鋒視窗：錯誤詳細資訊、建議的解決步驟，以及疑難排解文件的連結。

![複寫狀態](./media/how-to-connect-health-adds/aadconnect-health-adds-replication.png)

## <a name="monitoring"></a>監視
這項功能提供不同效能計數器的圖形化趨勢，這是持續從每個受監視的網域控制站收集而來。 網域控制站的效能可以輕鬆地和樹系中所有其他受監視的網域控制站進行比較。 此外，您可以看到各種效能計數器並排，在環境中針對問題進行疑難排解時這很有用。

![監視](./media/how-to-connect-health-adds/aadconnect-health-adds-monitoring.png)

根據預設，已預先選取四個效能計數器；不過，您可以按篩選命令並選取或取消選取任何想要的效能計數器。 此外，您可以連按兩下效能計數器圖形以開啟新的刀鋒視窗，其中包含每個受監視網域控制站的資料點。

## <a name="related-links"></a>相關連結
* [Azure AD Connect Health](whatis-hybrid-identity-health.md)
* [Azure AD Connect Health 代理程式安裝](how-to-connect-health-agent-install.md)
* [Azure AD Connect Health 操作](how-to-connect-health-operations.md)
* [使用 Azure AD Connect Health 來搭配 AD FS](how-to-connect-health-adfs.md)
* [使用 Azure AD Connect Health 進行同步處理](how-to-connect-health-sync.md)
* [Azure AD Connect Health 常見問題集](reference-connect-health-faq.md)
* [Azure AD Connect Health 版本歷程記錄](reference-connect-health-version-history.md)

