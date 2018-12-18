---
title: 已編製索引的 Azure SaaS SQL 應用程式影片 | Microsoft Docs
description: 本文就 2017 年 10 月 11 日所舉辦之 Ignite 會議有關 SaaS DB 租用應用程式設計的 81 分鐘影片，編製各個時間點的索引。 您可以直接跳到感興趣的部分。 至少說明 3 個模式。 說明可簡化開發和管理的 Azure 功能。
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: MightyPen
ms.author: genemi
ms.reviewer: billgib
manager: craigg
ms.date: 05/14/2018
ms.openlocfilehash: b472baf77cc8f253337aa0780321a6e7dd3fecac
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056676"
---
# <a name="video-indexed-and-annotated-for-multi-tenant-saas-app-using-azure-sql-database"></a>影片索引與註解，適用主題為使用 Azure SQL Database 的多租用戶 SaaS 應用程式之影片

就主題為 SaaS 租用模型或模式的影片，本文已將其 81 分鐘的片長編製成數個時間點的索引，並附上註解。 本文可讓您向後或向前跳到影片中感興趣的部分。 影片說明 Azure SQL Database 上多租用戶資料庫應用程式的主要設計選項。 該影片包括示範、管理程式碼的逐步解說，也會不時詳談書面文件中不曾有過的經驗。

影片詳述以下書面文件中的資訊： 
- *概念：*[多租用戶 SaaS 資料庫租用模式][saas-concept-design-patterns-563e]
- *教學課程：*[Wingtip Tickets SaaS 應用程式][saas-how-welcome-wingtip-app-679t]

影片與文章說明在雲端的 Azure SQL Database 上建立多租用戶應用程式的許多階段。 Azure SQL Database 的特殊功能可更輕鬆地開發和實作多租用戶應用程式，同時易於管理且兼顧效能。

我們會定期更新書面文件。 影片不會再經編輯或更新，因此最後其詳細資料多半會過期。



## <a name="sequence-of-38-time-indexed-screenshots"></a>38 個時間索引的螢幕擷取畫面順序

本節為長達 81 分鐘影片內的 38 項討論主題編製時間位置的索引。 每個時間索引都會標註一個影片的螢幕擷取畫面，有時會含有其他資訊。

每個時間索引的格式都是 *h:mm: ss*。 例如，編製的第二個時間位置索引標示為**會議目標**，開始的時間位置大約是 **0:03:11**。


### <a name="compact-links-to-video-indexed-time-locations"></a>編製影片時間位置索引的簡短連結

下列標題是本文稍後其對應標註章節的連結：

- [1.**(開始)** 歡迎投影片，0:00:03](#anchor-image-wtip-min00001)
- [2.會議目標，0:03:11](#anchor-image-wtip-min00311)
- [3.議程，0:04:17](#anchor-image-wtip-min00417)
- [4.多租用戶 Web 應用程式，0:05:05](#anchor-image-wtip-min00505)
- [5.作用中的應用程式 Web 表單，0:05:55](#anchor-image-wtip-min00555)
- [6.每個租用戶成本 (擴展、隔離、復原)，0:09:31](#anchor-image-wtip-min00931)
- [7.多租用戶的資料庫模型：優缺點，0:11:59](#anchor-image-wtip-min01159)
- [8.混合式模型兼具 MT/ST 優點，0:13:01](#anchor-image-wtip-min01301)
- [9.單一租用戶與多租用戶：優缺點，0:16:44](#anchor-image-wtip-min01644)
- [10.集區對無法預測的工作負載而言符合成本效益，0:19:36](#anchor-image-wtip-min01936)
- [11.每個租用戶的資料庫和混合式 ST/MT 示範，0:20:08](#anchor-image-wtip-min02008)
- [12.顯示 Dojo 的即時應用程式表單，0:20:29](#anchor-image-wtip-min02029)
- [13.MYOB 並非 DBA 深入解析，0:28:54](#anchor-image-wtip-min02854)
- [14.MYOB 彈性集區使用範例，0:29:40](#anchor-image-wtip-min02940)
- [15.向 MYOB 和其他 ISV 學習，0:31:36](#anchor-image-wtip-min03136)
- [16.撰寫成 E2E SaaS 案例的模式，0:43:15](#anchor-image-wtip-min04315)
- [17.Canonical 混合式多租用戶 SaaS 應用程式，0:47:33](#anchor-image-wtip-min04733)
- [18.Wingtip SaaS 範例應用程式，0:48:10](#anchor-image-wtip-min04810)
- [19.在教學課程中探索的案例和模式，0:49:10](#anchor-image-wtip-min04910)
- [20.教學課程與 Github 存放庫示範，0:50:18](#anchor-image-wtip-min05018)
- [21.Github 存放庫 Microsoft/WingtipSaaS，0:50:38](#anchor-image-wtip-min05038)
- [22.探索模式，0:56:20](#anchor-image-wtip-min05620)
- [23.佈建租用戶並上線，0:57:44](#anchor-image-wtip-min05744)
- [24.佈建租用戶和應用程式連線，0:58:58](#anchor-image-wtip-min05858)
- [25.管理指令碼佈建單一租用戶示範，0:59:43](#anchor-image-wtip-min05943)
- [26.用於佈建和目錄的 PowerShell，1:00:02](#anchor-image-wtip-min10002)
- [27.T-SQL SELECT * FROM TenantsExtended，1:03:30](#anchor-image-wtip-min10330)
- [28.管理無法預測的租用戶工作負載，1:04:36](#anchor-image-wtip-min10436)
- [29.彈性集區監視，1:06:39](#anchor-image-wtip-min10639)
- [30.負載產生與效能監視，1:09:42](#anchor-image-wtip-min10942)
- [31.大規模結構描述管理，1:10:33](#anchor-image-wtip-min11033)
- [32.跨租用戶資料庫的分散式查詢，1:12:21](#anchor-image-wtip-min11221)
- [33.票證產生示範，1:12:32](#anchor-image-wtip-min11232)
- [34.SSMS 臨機操作分析，1:12:46](#anchor-image-wtip-min11246)
- [35.將租用戶資料擷取至 SQL DW，1:16:32](#anchor-image-wtip-min11632)
- [36.每日銷售資料分佈圖表，1:16:48](#anchor-image-wtip-min11648)
- [37.總結與呼籲，1:19:52](#anchor-image-wtip-min11952)
- [38.可取得詳細資訊的資源，1:20:42](#anchor-image-wtip-min12042)


&nbsp;

### <a name="annotated-index-time-locations-in-the-video"></a>影片中標註的時間位置索引

按一下任何螢幕擷取畫面影像，即可前往影片中確切的時間位置。


&nbsp; <a name="anchor-image-wtip-min00001"/>
#### <a name="1-start-welcome-slide-00001"></a>1.*(開始)* 歡迎投影片，0:00:01

*向 MYOB 學習：適用於 Azure SQL Database 上 SaaS 應用程式的設計模式 - BRK3120*

[![歡迎投影片][image-wtip-min00003-brk3120-whole-welcome]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1)

- 標題：向 MYOB 學習：適用於 Azure SQL Database 上 SaaS 應用程式的設計模式
- Bill.Gibson@microsoft.com
- 程式總經理，Azure SQL Database
- 2017 年 10 月 11 日美國佛羅里達州奧蘭多市，Microsoft Ignite 會議 BRK3120


&nbsp; <a name="anchor-image-wtip-min00311"/>
#### <a name="2-session-objectives-00153"></a>2.會議目標，0:01:53
[![會議目標][image-wtip-min00311-session]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=113)

- 多租用戶應用程式替代模型的優缺點。
- 可降低開發、管理和資源成本的 SaaS 模式。
- 範例應用程式 + 指令碼。
- PaaS 功能 + SaaS 模式讓 SQL Database 成為具高擴充性、符合成本效益的資料平台，適合多租用戶 SaaS 使用。


&nbsp; <a name="anchor-image-wtip-min00417"/>
#### <a name="3-agenda-00409"></a>3.議程，0:04:09
[![議程][image-wtip-min00417-agenda]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=249)


&nbsp; <a name="anchor-image-wtip-min00505"/>
#### <a name="4-multi-tenant-web-app-00500"></a>4.多租用戶 Web 應用程式，0:05:00
[![Wingtip SaaS 應用程式：多租用戶 Web 應用程式][image-wtip-min00505-web-app]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=300)


&nbsp; <a name="anchor-image-wtip-min00555"/>
#### <a name="5-app-web-form-in-action-00539"></a>5.作用中的應用程式 Web 表單，0:05:39
[![作用中的應用程式 Web 表單][image-wtip-min00555-app-web-form]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=339)


&nbsp; <a name="anchor-image-wtip-min00931"/>
#### <a name="6-per-tenant-cost-scale-isolation-recovery-00658"></a>6.每個租用戶成本 (擴展、隔離、復原)，0:06:58
[![每個租用戶成本、擴展、隔離、復原][image-wtip-min00931-per-tenant-cost]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=418)


&nbsp; <a name="anchor-image-wtip-min01159"/>
#### <a name="7-database-models-for-multi-tenant-pros-and-cons-00952"></a>7.多租用戶的資料庫模型：優缺點，0:09:52
[![多租用戶的資料庫模型：優缺點][image-wtip-min01159-db-models-pros-cons]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=592)


&nbsp; <a name="anchor-image-wtip-min01301"/>
#### <a name="8-hybrid-model-blends-benefits-of-mtst-01229"></a>8.混合式模型兼具 MT/ST 優點，0:12:29
[![混合式模型兼具 MT/ST 優點][image-wtip-min01301-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=749)


&nbsp; <a name="anchor-image-wtip-min01644"/>
#### <a name="9-single-tenant-vs-multi-tenant-pros-and-cons-01311"></a>9.單一租用戶與多租用戶：優缺點，0:13:11
[![單一租用戶與多租用戶：優缺點][image-wtip-min01644-st-vs-mt]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=791)


&nbsp; <a name="anchor-image-wtip-min01936"/>
#### <a name="10-pools-are-cost-effective-for-unpredictable-workloads-01749"></a>10.集區對無法預測的工作負載而言符合成本效益，0:17:49
[![集區對無法預測的工作負載而言符合成本效益][image-wtip-min01936-pools-cost]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1069)


&nbsp; <a name="anchor-image-wtip-min02008"/>
#### <a name="11-demo-of-database-per-tenant-and-hybrid-stmt-01959"></a>11.每個租用戶的資料庫和混合式 ST/MT 示範，0:19:59
[![每個租用戶的資料庫和混合式 ST/MT 示範][image-wtip-min02008-demo-st-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1199)


&nbsp; <a name="anchor-image-wtip-min02029"/>
#### <a name="12-live-app-form-showing-dojo-02010"></a>12.顯示 Dojo 的即時應用程式表單，0:20:10
[![顯示 Dojo 的即時應用程式表單][image-wtip-min02029-live-app-form-dojo]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1210)

&nbsp; <a name="anchor-image-wtip-min02854"/>
#### <a name="13-myob-and-not-a-dba-in-sight-02506"></a>13.MYOB 並非 DBA 深入解析，0:25:06
[![MYOB 並非 DBA 深入解析][image-wtip-min02854-myob-no-dba]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1506)


&nbsp; <a name="anchor-image-wtip-min02940"/>
#### <a name="14-myob-elastic-pool-usage-example-02930"></a>14.MYOB 彈性集區使用範例，0:29:30
[![MYOB 彈性集區使用範例][image-wtip-min02940-myob-elastic]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1770)


&nbsp; <a name="anchor-image-wtip-min03136"/>
#### <a name="15-learning-from-myob-and-other-isvs-03125"></a>15.向 MYOB 和其他 ISV 學習，0:31:25
[![向 MYOB 和其他 ISV 學習][image-wtip-min03136-learning-isvs]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1885)


&nbsp; <a name="anchor-image-wtip-min04315"/>
#### <a name="16-patterns-compose-into-e2e-saas-scenario-03142"></a>16.撰寫成 E2E SaaS 案例的模式，0:31:42
[![撰寫成 E2E SaaS 案例的模式][image-wtip-min04315-patterns-compose]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1902)


&nbsp; <a name="anchor-image-wtip-min04733"/>
#### <a name="17-canonical-hybrid-multi-tenant-saas-app-04604"></a>17.Canonical 混合式多租用戶 SaaS 應用程式，0:46:04
[![Canonical 混合式多租用戶 SaaS 應用程式][image-wtip-min04733-canonical-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2764)


&nbsp; <a name="anchor-image-wtip-min04810"/>
#### <a name="18-wingtip-saas-sample-app-04801"></a>18.Wingtip SaaS 範例應用程式，0:48:01
[![Wingtip SaaS 範例應用程式][image-wtip-min04810-wingtip-saas-app]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2881)


&nbsp; <a name="anchor-image-wtip-min04910"/>
#### <a name="19-scenarios-and-patterns-explored-in-the-tutorials-04900"></a>19.在教學課程中探索的案例和模式，0:49:00
[![在教學課程中探索的案例和模式][image-wtip-min04910-scenarios-tutorials]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2940)


&nbsp; <a name="anchor-image-wtip-min05018"/>
#### <a name="20-demo-of-tutorials-and-github-repository-05012"></a>20.教學課程與 Github 存放庫示範，0:50:12
[![教學課程與 Github 存放庫示範][image-wtip-min05018-demo-tutorials-github]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3012)


&nbsp; <a name="anchor-image-wtip-min05038"/>
#### <a name="21-github-repo-microsoftwingtipsaas-05032"></a>21.Github 存放庫 Microsoft/WingtipSaaS，0:50:32
[![Github 存放庫 Microsoft/WingtipSaaS][image-wtip-min05038-github-wingtipsaas]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3032)


&nbsp; <a name="anchor-image-wtip-min05620"/>
#### <a name="22-exploring-the-patterns-05615"></a>22.探索模式，0:56:15
[![探索模式][image-wtip-min05620-exploring-patterns]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3375)


&nbsp; <a name="anchor-image-wtip-min05744"/>
#### <a name="23-provisioning-tenants-and-onboarding-05619"></a>23.佈建租用戶並上線，0:56:19
[![佈建租用戶並上線][image-wtip-min05744-provisioning-tenants-onboarding-1]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3379)


&nbsp; <a name="anchor-image-wtip-min05858"/>
#### <a name="24-provisioning-tenants-and-application-connection-05752"></a>24.佈建租用戶和應用程式連線，0:57:52
[![佈建租用戶和應用程式連線][image-wtip-min05858-provisioning-tenants-app-connection-2]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3472)


&nbsp; <a name="anchor-image-wtip-min05943"/>
#### <a name="25-demo-of-management-scripts-provisioning-a-single-tenant-05936"></a>25.管理指令碼佈建單一租用戶示範，0:59:36
[![管理指令碼佈建單一租用戶示範][image-wtip-min05943-demo-management-scripts-st]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3576)


&nbsp; <a name="anchor-image-wtip-min10002"/>
#### <a name="26-powershell-to-provision-and-catalog-05956"></a>26.用於佈建和目錄的 PowerShell，0:59:56
[![用於佈建和目錄的 PowerShell][image-wtip-min10002-powershell-provision-catalog]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3596)


&nbsp; <a name="anchor-image-wtip-min10330"/>
#### <a name="27-t-sql-select--from-tenantsextended-10325"></a>27.T-SQL SELECT * FROM TenantsExtended，1:03:25
[![T-SQL SELECT * FROM TenantsExtended][image-wtip-min10330-sql-select-tenantsextended]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3805)


&nbsp; <a name="anchor-image-wtip-min10436"/>
#### <a name="28-managing-unpredictable-tenant-workloads-10334"></a>28.管理無法預測的租用戶工作負載，1:03:34
[![管理無法預測的租用戶工作負載][image-wtip-min10436-managing-unpredictable-workloads]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3814)


&nbsp; <a name="anchor-image-wtip-min10639"/>
#### <a name="29-elastic-pool-monitoring-10632"></a>29.彈性集區監視，1:06:32
[![彈性集區監視][image-wtip-min10639-elastic-pool-monitoring]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3992)


&nbsp; <a name="anchor-image-wtip-min10942"/>
#### <a name="30-load-generation-and-performance-monitoring-10937"></a>30.負載產生與效能監視，1:09:37
[![負載產生與效能監視][image-wtip-min10942-load-generation]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4117)


&nbsp; <a name="anchor-image-wtip-min11033"/>
#### <a name="31-schema-management-at-scale-10940"></a>31.大規模結構描述管理，1:09:40
[![大規模結構描述管理][image-wtip-min11033-schema-management-scale]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=34120)


&nbsp; <a name="anchor-image-wtip-min11221"/>
#### <a name="32-distributed-query-across-tenant-databases-11118"></a>32.跨租用戶資料庫的分散式查詢，1:11:18
[![跨租用戶資料庫的分散式查詢][image-wtip-min11221-distributed-query]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4278)


&nbsp; <a name="anchor-image-wtip-min11232"/>
#### <a name="33-demo-of-ticket-generation-11228"></a>33.票證產生示範，1:12:28
[![票證產生示範][image-wtip-min11232-demo-ticket-generation]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4348)


&nbsp; <a name="anchor-image-wtip-min11246"/>
#### <a name="34-ssms-adhoc-analytics-11235"></a>34.SSMS 臨機操作分析，1:12:35
[![SSMS 臨機操作分析][image-wtip-min11246-ssms-adhoc-analytics]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4355)


&nbsp; <a name="anchor-image-wtip-min11632"/>
#### <a name="35-extract-tenant-data-into-sql-dw-11546"></a>35.將租用戶資料擷取至 SQL DW，1:15:46
[![將租用戶資料擷取至 SQL DW][image-wtip-min11632-extract-tenant-data-sql-dw]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4546)


&nbsp; <a name="anchor-image-wtip-min11648"/>
#### <a name="36-graph-of-daily-sale-distribution-11638"></a>36.每日銷售資料分佈圖表，1:16:38
[![每日銷售資料分佈圖表][image-wtip-min11648-graph-daily-sale-distribution]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4598)


&nbsp; <a name="anchor-image-wtip-min11952"/>
#### <a name="37-wrap-up-and-call-to-action-11743"></a>37.總結與呼籲，1:17:43
[![總結與呼籲][image-wtip-min11952-wrap-up-call-action]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4663)


&nbsp; <a name="anchor-image-wtip-min12042"/>
#### <a name="38-resources-for-more-information-12035"></a>38.可取得詳細資訊的資源，1:20:35
[![可取得詳細資訊的資源][image-wtip-min12042-resources-more-info]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4835)

- [部落格文章，2017 年 5 月 22 日][resource-blog-saas-patterns-app-dev-sql-db-768h]

- *概念：*[多租用戶 SaaS 資料庫租用模式][saas-concept-design-patterns-563e]

- *教學課程：*[Wingtip Tickets SaaS 應用程式][saas-how-welcome-wingtip-app-679t]

- Wingtip 票證 SaaS 租用戶應用程式的 Github 存放庫：
    - [Github 存放庫：獨立應用程式模型][github-wingtip-standaloneapp]。
    - [Github 存放庫：每一租用戶一個資料庫模型][github-wingtip-dbpertenant]。
    - [Github 存放庫：多租用戶資料庫模型][github-wingtip-multitenantdb]。





## <a name="next-steps"></a>後續步驟

- [第一堂教學課程文章][saas-how-welcome-wingtip-app-679t]




<!-- Image link reference IDs. -->

[image-wtip-min00003-brk3120-whole-welcome]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00003-brk3120-welcome-myob-design-saas-app-sql-db.png "歡迎投影片"

[image-wtip-min00311-session]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00311-session-objectives-takeaway.png "會議目標"

[image-wtip-min00417-agenda]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00417-agenda-app-management-models-patterns.png "議程"

[image-wtip-min00505-web-app]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00505-wingtip-saas-app-mt-web.png "Wingtip SaaS 應用程式：多租用戶 Web 應用程式"

[image-wtip-min00555-app-web-form]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00555-app-form-contoso-concert-hall-night-opera.png "作用中的應用程式 Web 表單"

[image-wtip-min00931-per-tenant-cost]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00931-saas-data-management-concerns.png "每個租用戶成本、擴展、隔離、復原"

[image-wtip-min01159-db-models-pros-cons]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01159-db-models-multi-tenant-saas-apps.png "多租用戶的資料庫模型：優缺點"

[image-wtip-min01301-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01301-hybrib-model-blends-benefits-mt-st.png "混合式模型兼具 MT/ST 優點"

[image-wtip-min01644-st-vs-mt]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01644-st-mt-pros-cons.png "單一租用戶與多租用戶：優缺點"

[image-wtip-min01936-pools-cost]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01936-pools-cost-effective-unpredictable-workloads.png "集區對無法預測的工作負載而言符合成本效益"

[image-wtip-min02008-demo-st-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02008-demo-st-hybrid.png "每個租用戶的資料庫和混合式 ST/MT 示範"

[image-wtip-min02029-live-app-form-dojo]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02029-app-form-dogwwod-dojo.png "顯示 Dojo 的即時應用程式表單"

[image-wtip-min02854-myob-no-dba]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02854-myob-no-dba.png "MYOB 並非 DBA 深入解析"

[image-wtip-min02940-myob-elastic]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02940-myob-elastic-pool-usage-example.png "MYOB 彈性集區使用範例"

[image-wtip-min03136-learning-isvs]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min03136-myob-isv-saas-patterns-design-scale.png "向 MYOB 和其他 ISV 學習"

[image-wtip-min04315-patterns-compose]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04315-patterns-compose-into-e2e-saas-scenario-st-mt.png "撰寫成 E2E SaaS 案例的模式"

[image-wtip-min04733-canonical-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04733-canonical-hybrid-mt-saas-app.png "Canonical 混合式多租用戶 SaaS 應用程式"

[image-wtip-min04810-wingtip-saas-app]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04810-saas-sample-app-descr-of-modules-links.png "Wingtip SaaS 範例應用程式"

[image-wtip-min04910-scenarios-tutorials]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04910-scenarios-patterns-explored-tutorials.png "在教學課程中探索的案例和模式"

[image-wtip-min05018-demo-tutorials-github]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05018-demo-saas-tutorials-github-repo.png "教學課程與 Github 存放庫示範"

[image-wtip-min05038-github-wingtipsaas]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05038-github-repo-wingtipsaas.png "Github 存放庫 Microsoft/WingtipSaaS"

[image-wtip-min05620-exploring-patterns]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05620-exploring-patterns-tutorials.png "探索模式"

[image-wtip-min05744-provisioning-tenants-onboarding-1]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05744-provisioning-tenants-connecting-run-time-1.png "佈建租用戶並上線"

[image-wtip-min05858-provisioning-tenants-app-connection-2]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05858-provisioning-tenants-connecting-run-time-2.png "佈建租用戶和應用程式連線"

[image-wtip-min05943-demo-management-scripts-st]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05943-demo-management-scripts-provisioning-st.png "管理指令碼佈建單一租用戶示範"

[image-wtip-min10002-powershell-provision-catalog]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10002-powershell-code.png "用於佈建和目錄的 PowerShell"

[image-wtip-min10330-sql-select-tenantsextended]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10330-ssms-tenantcatalog.png "T-SQL SELECT * FROM TenantsExtended"

[image-wtip-min10436-managing-unpredictable-workloads]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10436-managing-unpredictable-tenant-workloads.png "管理無法預測的租用戶工作負載"

[image-wtip-min10639-elastic-pool-monitoring]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10639-elastic-pool-monitoring.png "彈性集區監視"

[image-wtip-min10942-load-generation]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10942-schema-management-scale.png "負載產生與效能監視"

[image-wtip-min11033-schema-management-scale]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11033-schema-manage-1000s-dbs-one.png "大規模結構描述管理"

[image-wtip-min11221-distributed-query]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11221-distributed-query-all-tenants-asif-single-db.png "跨租用戶資料庫的分散式查詢"

[image-wtip-min11232-demo-ticket-generation]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11232-demo-ticket-generation-distributed-query.png "票證產生示範"

[image-wtip-min11246-ssms-adhoc-analytics]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11246-tsql-adhoc-analystics-db-elastic-query.png "SSMS 臨機操作分析"

[image-wtip-min11632-extract-tenant-data-sql-dw]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11632-extract-tenant-data-analytics-db-dw.png "將租用戶資料擷取至 SQL DW"

[image-wtip-min11648-graph-daily-sale-distribution]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11648-graph-daily-sale-contoso-concert-hall.png "每日銷售資料分佈圖表"

[image-wtip-min11952-wrap-up-call-action]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11952-wrap-call-action-saasfeedback.png "總結與呼籲"

[image-wtip-min12042-resources-more-info]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min12042-resources-blog-github-tutorials-get-started.png "可取得詳細資訊的資源"




<!-- Article link reference IDs. -->

[saas-concept-design-patterns-563e]: saas-tenancy-app-design-patterns.md

[saas-how-welcome-wingtip-app-679t]: saas-tenancy-welcome-wingtip-tickets-app.md


[video-on-youtube-com-478y]: https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1

[video-on-channel9-479c]: https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK3120


[resource-blog-saas-patterns-app-dev-sql-db-768h]: https://azure.microsoft.com/blog/saas-patterns-accelerate-saas-application-development-on-sql-database/


[github-wingtip-standaloneapp]: https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp/

[github-wingtip-dbpertenant]: https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/

[github-wingtip-multitenantdb]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDB/

