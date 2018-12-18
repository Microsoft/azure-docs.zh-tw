---
title: 在 BizTalk 服務中建立和還原備份 | Microsoft Docs
description: BizTalk 服務包含備份與還原功能。 了解如何建立和還原備份，以及判斷該備份什麼。 MABS，WABS
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: anneta
editor: ''
ms.assetid: 59f91173-4683-48df-abd5-41262bfce6df
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/07/2016
ms.author: mandia
ms.openlocfilehash: 90cf2d0ddbba47a856bf1299a101c5185873b5d8
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39214407"
---
# <a name="biztalk-services-backup-and-restore"></a>BizTalk 服務：備份與還原

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

Azure BizTalk 服務包含備份與還原功能。 

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

> [!NOTE]
> 混合式連接無法備份，與版本無關。 您必須重新建立混合式連接。


## <a name="before-you-begin"></a>開始之前
* 備份與還原可能不適用於部分版本。 請參閱「 [BizTalk 服務：版本圖表](biztalk-editions-feature-chart.md)」(英文)。
* 備份內容可以還原至相同的 BizTalk 服務，或還原至新的 BizTalk 服務。 若要使用相同名稱來還原 BizTalk 服務，必須刪除現有的 BizTalk 服務，且名稱必須可用。 刪除 BizTalk 服務之後，可能需要較長的時間，才能使用相同的名稱。 如果無法等待相同的名稱可用，請還原至新的 BizTalk 服務。
* BizTalk 服務可以還原至相同版本或更高的版本。 從建立備份之後，不支援將 BizTalk 服務還原至較低版本。
  
    例如，使用基本版本的備份可以還原至高級版本。 使用高級版本的備份不能還原至標準版本。
* EDI 控制編號會備份以維持控制編號的連貫性。 如果在上次備份之後處理訊息，則還原此備份內容會產生重複的控制編號。
* 如果批次中有作用中的訊息，請在執行備份前 **先** 處理批次。 無論是建立隨選備份或排定備份，都不會儲存批次中的訊息。 
  
    **建立備份時，如果批次中有作用中的訊息，則不會備份這些訊息，且這些訊息將會遺失。**
* 選用：在 BizTalk 服務入口網站中，停止任何管理作業。

## <a name="create-a-backup"></a>建立備份
您隨時都可以建立備份，完全決由掌控。 若要建立備份，請使用[用於管理 Azure 上之 BizTalk 服務的 REST API](https://msdn.microsoft.com/library/azure/dn232347.aspx)。

## <a name="restore"></a>Restore
若要還原備份，請使用[用於管理 Azure 上之 BizTalk 服務的 REST API](https://msdn.microsoft.com/library/azure/dn232347.aspx)。

### <a name="postrestore"></a>還原備份之後
BizTalk 服務永遠還原成 **暫止** 狀態。 在此狀態下，您可以在新環境開始運作前進行任何設定變更，包括：

* 若您使用 Azure BizTalk 服務 SDK 來建立 BizTalk 服務應用程式，您可能需要在那些應用程式中更新存取控制 (ACS) 認證，才能搭配還原後的環境使用。
* 您可以還原 BizTalk 服務來複寫現有的 BizTalk 服務環境。 在此情況下，如果在原始 BizTalk 服務入口網站中有設定的協議使用 FTP 來源資料夾，則您可能需要在剛還原的環境中更新協議，以使用其他 FTP 來源資料夾。 否則，可能會有兩個不同的協議都嘗試提取相同的訊息。
* 如果您已進行還原而產生多個 BizTalk 服務環境，則在使用 Visual Studio 應用程式、PowerShell Cmdlet、REST API 或交易夥伴管理 OM API 時，請確定目標環境正確。
* 在剛還原的 BizTalk 服務環境上，建議設定自動備份。

## <a name="what-gets-backed-up"></a>備份什麼項目
建立備份時會備份下列項目：

<table border="1"> 
<tr bgcolor="FAF9F9">
<th> </th>
<TH>備份的項目</TH> 
</tr> 
<tr>
<td colspan="2">
 <strong>Azure BizTalk 服務入口網站</strong></td>
</tr> 
<tr>
<td>組態和執行階段</td> 
<td>
<ul>
<li>夥伴和設定檔詳細資料</li>
<li>夥伴協議</li>
<li>已部署的自訂組件</li>
<li>已部署的橋接器</li>
<li>憑證</li>
<li>已部署的轉換</li>
<li>管線</li>
<li>BizTalk 服務入口網站中建立和儲存的範本</li>
<li>X12 ST01 和 GS01 對應</li>
<li>控制編號 (EDI)</li>
<li>AS2 訊息 MIC 值</li>
</ul>
</td>
</tr> 

<tr>
<td colspan="2">
 <strong>Azure BizTalk 服務</strong></td>
</tr> 
<tr>
<td>SSL 憑證</td> 
<td>
<ul>
<li>SSL 憑證資料</li>
<li>SSL 憑證密碼</li>
</ul>
</td>
</tr> 
<tr>
<td>BizTalk 服務設定</td> 
<td>
<ul>
<li>調整單位計數</li>
<li>版本</li>
<li>產品版本</li>
<li>區域/資料中心</li>
<li>存取控制服務 (ACS) 命名空間和金鑰</li>
<li>追蹤資料庫連接字串</li>
<li>封存儲存體帳戶連接字串</li>
<li>監視儲存體帳戶連接字串</li>
</ul>
</td>
</tr> 
<tr>
<td colspan="2">
 <strong>其他項目</strong></td>
</tr> 
<tr>
<td>追蹤資料庫</td> 
<td>建立 BizTalk 服務時需要輸入追蹤資料庫詳細資料，包括 Azure SQL Database 伺服器和追蹤資料庫名稱。 不會自動備份追蹤資料庫。
<br/><br/>
<strong>重要</strong><br/>
如果刪除了追蹤資料庫，且需要復原資料庫，先前的備份必須存在。 如果備份不存在，則無法復原追蹤資料庫及其資料。 在此情況下，請以相同的資料庫名稱建立新的追蹤資料庫。 建議採用地理複寫。</td>
</tr> 
</table>

## <a name="next"></a>下一頁
若要建立 Azure BizTalk 服務，請移至 [BizTalk 服務：佈建](http://go.microsoft.com/fwlink/p/?LinkID=302280)。 若要開始建立應用程式，請移至 [Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=235197)(英文)。

## <a name="see-also"></a>另請參閱
* [備份 BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=325584)
* [從備份還原 BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=325582)
* [BizTalk 服務：開發人員、基本、標準和高級版本圖表](http://go.microsoft.com/fwlink/p/?LinkID=302279)
* [BizTalk 服務：佈建](http://go.microsoft.com/fwlink/p/?LinkID=302280)
* [BizTalk 服務：佈建狀態圖](http://go.microsoft.com/fwlink/p/?LinkID=329870)
* [BizTalk 服務：儀表板、監視和調整索引標籤](http://go.microsoft.com/fwlink/p/?LinkID=302281)
* [BizTalk 服務：節流](http://go.microsoft.com/fwlink/p/?LinkID=302282)
* [BizTalk 服務：簽發者名稱和簽發者金鑰](http://go.microsoft.com/fwlink/p/?LinkID=303941)
* [如何開始使用 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=302335)

[BackupStatus]: ./media/biztalk-backup-restore/status-last-backup.png
[Restore]: ./media/biztalk-backup-restore/restore-ui.png
[AutomaticBU]: ./media/biztalk-backup-restore/AutomaticBU.png
[RestoreBizTalkService]: ./media/biztalk-backup-restore/RestoreBizTalkServiceWindow.png

