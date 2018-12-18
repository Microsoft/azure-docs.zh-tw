---
title: 在 Azure 入口網站中設定適用於 PostgreSQL 的 Azure 資料庫的計量警示
description: 本文描述如何從 Azure 入口網站，設定和存取「適用於 PostgreSQL 的 Azure 資料庫」的計量警示。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: b4b15998276dd6c32e9c15622aa0251c6c066085
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
ms.locfileid: "29690248"
---
# <a name="use-the-azure-portal-to-set-up-alerts-on-metrics-for-azure-database-for-postgresql"></a>使用 Azure 入口網站來設定適用於 PostgreSQL 之 Azure 資料庫的計量警示 

本文說明如何使用 Azure 入口網站來設定「適用於 PostgreSQL 的 Azure 資料庫」警示。 您可以收到以 Azure 服務的監視計量為基礎的警示。

當所指定計量的值超出您指派的臨界值時，就會觸發警示。 不論是一開始符合條件時，還是之後不再符合條件時，都會觸發警示。 

您可以設定讓警示在觸發時執行下列動作︰
* 傳送電子郵件通知給服務管理員和共同管理員。
* 傳送電子郵件給您指定的其他電子郵件。
* 呼叫 Webhook。

您可以透過下列方式，來設定及取得警示規則的相關資訊：
* [Azure 入口網站](../monitoring-and-diagnostics/insights-alerts-portal.md)
* [PowerShell](../monitoring-and-diagnostics/insights-alerts-powershell.md)
* [命令列介面 (CLI)](../monitoring-and-diagnostics/insights-alerts-command-line-interface.md)
* [Azure 監視器 REST API](https://msdn.microsoft.com/library/azure/dn931945.aspx)

## <a name="create-an-alert-rule-on-a-metric-from-the-azure-portal"></a>從 Azure 入口網站建立計量的警示規則
1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您想要監視的「適用於 PostgreSQL 的 Azure 資料庫」伺服器。

2. 在資訊看板的 [監視] 區段底下，選取 [警示規則]，如下所示：

   ![選取 [警示規則]](./media/howto-alert-on-metric/1-alert-rules.png)

3. 選取 [新增計量警示] (+ 圖示)。 

4. [新增規則] 頁面隨即開啟，如下所示。  填寫必要資訊：

   ![[新增計量警示] 表單](./media/howto-alert-on-metric/2-add-rule-form.png)

   | 設定 | 說明  |
   |---------|---------|
   | Name | 提供警示規則的名稱。 在警示通知電子郵件中會傳送此值。 |
   | 說明 | 提供警示規則的簡短描述。 在警示通知電子郵件中會傳送此值。 |
   | 警示在 | 針對這類警示，請選擇 [計量]。 |
   | 訂用帳戶 | 此欄位會預先填入裝載「適用於 PostgreSQL 的 Azure 資料庫」的訂用帳戶。 |
   | 資源群組 | 此欄位會預先填入「適用於 PostgreSQL 的 Azure 資料庫」的資源群組。 |
   | 資源 | 此欄位會預先填入「適用於 PostgreSQL 的 Azure 資料庫」的名稱。 |
   | 計量 | 選取您要為其發出警示的計量。 例如 [儲存體百分比]。 |
   | 條件 | 選擇要用來比較計量的條件。 例如 [大於]。 |
   | 閾值 | 計量的臨界值，例如 85 (百分比)。 |
   | 期間 | 觸發警示之前，必須滿足計量規則的一段時間。 例如 [在過去 30 分鐘內]。 |

   根據此範例，警示會尋找儲存體百分比在 30 分鐘期間高於 85% 的情況。 當平均儲存體百分比高於 85% 達 30 分鐘時，就會觸發該警示。 發生第一次觸發之後，當平均儲存體百分比在 30 分鐘期間低於 85% 時，就會再次觸發該警示。

5. 選擇要用於警示規則的通知方法。 

   如果您想要在警示引發時傳送電子郵件給訂用帳戶管理員和共同管理員，請勾選 [將電子郵件寄給擁有者、參與者及讀者] 選項。

   如果您想要讓其他電子郵件信箱在警示引發時收到通知，請在 [其他管理員的電子郵件] 欄位新增它們。 以分號分隔多個電子郵件 - *email@contoso.com;email2@contoso.com*

   如果您想要在警示引發時呼叫 Webhook，可選擇在 [Webhook] 欄位中提供有效的 URI。

6. 選取 [確定] 可建立警示。

   在幾分鐘之內，警示會開始作用，且先前所述觸發。

## <a name="manage-your-alerts"></a>管理警示
建立警示之後，您可以選取它，然後進行下列動作：

* 檢視圖表，其中顯示與此警示相關的計量臨界值及前一天的實際值。
* **編輯**或**刪除**警示規則。
* 如果您想要暫時停止或恢復接收通知，可以將警示**停用**或**啟用**。

## <a name="next-steps"></a>後續步驟
* 深入了解 [在警示中設定 webhook](../monitoring-and-diagnostics/insights-webhooks-alerts.md)。
* 依照 [計量集合概觀](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md) 中的做法，確保您的服務可使用且有回應。
