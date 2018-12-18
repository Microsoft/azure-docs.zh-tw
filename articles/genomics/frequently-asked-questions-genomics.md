---
title: Microsoft Genomics：常見問題 | Microsoft Docs
titleSuffix: Azure
description: 有關 Microsoft Genomics 之客戶常見問題的解答。
services: microsoft-genomics
author: grhuynh
manager: jhubbard
editor: jasonwhowell
ms.author: grhuynh
ms.service: microsoft-genomics
ms.workload: genomics
ms.topic: article
ms.date: 12/07/2017
ms.openlocfilehash: b3f30dc5d185615370a8273f71554f784d286cd9
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31517013"
---
# <a name="microsoft-genomics-common-questions"></a>Microsoft Genomics：常見問題

本文會列出關於 Microsoft Genomics 的最常見疑問。 如需有關 Microsoft Genomics 服務的詳細資訊，請參閱[什麼是 Microsoft Genomics？](overview-what-is-genomics.md)。 如需有關疑難排解的詳細資訊，請參閱[疑難排解指南](troubleshooting-guide-genomics.md)。 


## <a name="what-is-the-sla-for-microsoft-genomics"></a>什麼是適用於 Microsoft Genomics 的 SLA？
我們保證 99.9% 的時間 Microsoft Genomics 服務都可用於接收工作流程 API 要求。 如需詳細資訊，請參閱 [SLA](https://azure.microsoft.com/support/legal/sla/genomics/v1_0/)。

## <a name="how-does-the-usage-of-microsoft-genomics-show-up-on-my-bill"></a>我的帳單上會用什麼方式顯示 Microsoft Genomics 的使用量？
Microsoft Genomics 是根據每個工作流程所處理的 Gb 數來計費。 如需詳細資訊，請參閱[定價](https://azure.microsoft.com/pricing/details/genomics/)。


## <a name="where-can-i-find-a-list-of-all-possible-commands-and-arguments-for-the-msgen-client"></a>哪裡可以找到 `msgen` 用戶端 的所有可能命令和引數清單？
執行 `msgen help` 即可取得完整的可用命令與引數清單。 如果未提供其他引數，它會顯示可用的說明區段清單，每個 `submit`、`list`、`cancel` 和 `status` 各有一個區段。 若要取得特定命令的說明，請輸入 `msgen help command`；例如，`msgen help submit` 可列出所有提交選項。

## <a name="what-are-the-most-commonly-used-commands-for-the-msgen-client"></a>`msgen` 用戶端的最常用命令有哪些？
最常用的命令是 `msgen` 用戶端的引數，包括： 

 |**命令**          |  **欄位描述** |
 |:--------------------|:-------------         |
 |`list`               |傳回您已提交的工作清單。 如需引數，請參閱 `msgen help list`。  |
 |`submit`             |將工作流程要求提交至服務。 如需引數，請參閱 `msgen help submit`。|
 |`status`             |傳回 `--workflow-id` 所指定的工作流程狀態。 另請參閱 `msgen help status`。 |
 |`cancel`             |傳送要求以取消 `--workflow-id` 所指定的工作流程處理。 另請參閱 `msgen help cancel`。 |

## <a name="where-do-i-get-the-value-for---api-url-base"></a>哪裡可取得 `--api-url-base` 的值？
移至 Azure 入口網站，然後開啟您的 Genomics 帳戶頁面。 在 [管理] 標題之下，選擇 [存取金鑰]。 您可以看到 API URL 以及存取金鑰。

## <a name="where-do-i-get-the-value-for---access-key"></a>哪裡可取得 `--access-key` 的值？
移至 Azure 入口網站，然後開啟您的 Genomics 帳戶頁面。 在 [管理] 標題之下，選擇 [存取金鑰]。 您可以看到 API URL 以及存取金鑰。

## <a name="why-do-i-need-two-access-keys"></a>為什麼需要兩個存取金鑰？
如果您想要在不中斷服務使用的情況下更新 (重新產生) 存取金鑰，您就需要兩個存取金鑰。 例如，您想要更新第一個金鑰。 在此情況下，您可以將所有新的工作流程切換成使用第二個金鑰。 然後，等到使用第一個金鑰的執行中工作流程完成為止。 然後才能更新金鑰。

## <a name="do-you-save-my-storage-account-keys"></a>您是否會儲存我的儲存體帳戶金鑰？
您的儲存體帳戶金鑰用來建立 Microsoft Genomics 服務的短期存取權杖，以讀取您的輸入檔案並寫入輸出檔案。 預設權杖持續時間為 48 小時。 使用 submit 命令的 `-sas/--sas-duration` 選項可以變更權杖持續時間；此值是以小時為單位。

## <a name="what-genome-references-can-i-use"></a>我可以使用哪些基因組參考？

支援以下參考：
 |參考              | `-pa/--process-args` 的值 |
 |:-------------         |:-------------                 |
 |b37                    | `R=b37m1`                     |
 |hg38                   | `R=hg38m1`                    |      
 |hg38 (沒有 alt 分析) | `R=hg38m1x`                   |  
 |hg19                   | `R=hg19m1`                    |    

## <a name="how-do-i-format-my-command-line-arguments-as-a-config-file"></a>如何將我的命令列引數格式化為組態檔？ 

msgen 了解以下列格式的組態檔：
* 所有選項都是以金鑰/值組的形式提供 (以冒號分隔其值與金鑰)。
空白字元會被忽略。
* 以 `#` 開頭的程式碼行會被忽略。
* 任何長格式的命令列引數都可以轉換成金鑰，方法是移除其前置連字號並以底線取代文字之間的連字號。 以下是一些轉換範例：

 |命令列引數            | 組態檔程式行 |
 |:-------------                   |:-------------                 |
 |`-u/--api-url-base https://url`  | *api_url_base:https://url*    |
 |`-k/--access-key KEY`            | *access_key:KEY*              |      
 |`-pa/--process-args R=B37m1`     | *process_args:R-b37m1*        |  

## <a name="next-steps"></a>後續步驟

使用下列資源來開始使用 Microsoft Genomics：
- 透過 Microsoft Genomics 服務開始執行您的第一個工作流程。 [透過 Microsoft Genomics 服務執行工作流程](quickstart-run-genomics-workflow-portal.md)
- 提交您自己的資料以供 Microsoft Genomics 服務處理：[配對的 FASTQ](quickstart-input-pair-FASTQ.md) | [BAM](quickstart-input-BAM.md) | [多重 FASTQ 或 BAM](quickstart-input-multiple.md) 

