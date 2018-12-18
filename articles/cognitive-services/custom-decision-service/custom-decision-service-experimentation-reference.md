---
title: 測試 - Azure 認知服務 | Microsoft Docs
description: 本文是 Azure 自訂決策服務測試的指南。
services: cognitive-services
author: marco-rossi29
manager: marco-rossi29
ms.service: cognitive-services
ms.topic: article
ms.date: 05/10/2018
ms.author: marossi
ms.openlocfilehash: b0ac0bc049d556423493f0c48dd9a548929bcd41
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370223"
---
# <a name="experimentation"></a>測試

自訂決策服務遵循 [Contextual Bandit (CB)](https://www.microsoft.com/en-us/research/blog/contextual-bandit-breakthrough-enables-deeper-personalization/) 的理論，重複觀察內容、採取動作，並觀察所選動作的獎勵。 內容個人化的一個範例是：內容描述使用者、動作是候選劇本，而獎勵測量使用者喜歡建議劇本的程度。

自訂決策服務會產生原則，因為它會從內容對應至動作。 使用特定目標原則，您想要知道其預期獎勵。 估計獎勵的一個方法是使用線上原則，並讓它選擇動作 (例如為使用者建議劇本)。 不過，這類線上評估的成本可能很高，原因有兩個：

* 它會向使用者公開未經測試的實驗性原則。
* 它不會調整規模以評估多個目標原則。

原則外 (Off-Policy) 評估是另一個範例。 若您有來自現有線上系統的記錄遵循記錄原則，原則外評估可以估計新目標原則的預期獎勵。

透過使用記錄檔，測試會嘗試找出具有最高估計預期獎勵的原則。 目標原則是由 [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki) 引數參數化。 在預設模式中，指令碼會測試各種 Vowpal Wabbit 引數，方法是將其附加至 `--base_command`。 此指令碼會執行下列動作：

* 從輸入檔的前 `--auto_lines` 行自動偵測功能命名空間。
* 對超參數 (`learning rate`、`L1 regularization` 和 `power_t`) 執行第一次掃掠。
* 測試原則評估 `--cb_type` (反向傾向分數 (`ips`) 或雙穩健 (`dr`))。 如需詳細資訊，請參閱 [Contextual Bandit 範例](https://github.com/JohnLangford/vowpal_wabbit/wiki/Contextual-Bandit-Example)。
* 測試臨界值。
* 測試二次方互動功能：
   * **暴力密碼破解階段**：使用 `--q_bruteforce_terms` 個或更少配對測試所有組合。
   * **窮盡階段**：新增最佳配對，直到 `--q_greedy_stop` 回合沒有任何改善。
* 對超參數 (`learning rate`、`L1 regularization` 和 `power_t`) 執行第二次掃掠。

控制這些步驟的參數包含一些 Vowpal Wabbit 引數：
- 範例操作選項：
  - 共用命名空間
  - 動作命名空間
  - 臨界命名空間
  - 二次方功能
- 更新規則選項
  - 學習速度
  - L1 正規化
  - t 乘冪值

如需上述引數的深入說明，請參閱 [Vowpal Wabbit command-line arguments](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments) (Vowpal Wabbit 命令列引數)。

## <a name="prerequisites"></a>先決條件
- Vowpal Wabbit：已安裝並在您的路徑上。
  - Windows：[使用 `.msi` 安裝程式](https://github.com/eisber/vowpal_wabbit/releases)。
  - 其他平台：[取得原始程式碼](https://github.com/JohnLangford/vowpal_wabbit/releases)。
- Python 3：已安裝並在您的路徑上。
- NumPy：使用您選擇的套件管理員。
- *Microsoft/mwt-ds* 存放庫：[複製存放庫](https://github.com/Microsoft/mwt-ds).
- 決策服務 JSON 記錄檔：根據預設，基本命令包括 `--dsjson`，這會啟用輸入資料檔案的決策服務 JSON 剖析。 [取得此格式的範例](https://github.com/JohnLangford/vowpal_wabbit/blob/master/test/train-sets/decisionservice.json)。

## <a name="usage"></a>使用量
移至 `mwt-ds/DataScience` 並使用相關引數執行 `Experimentation.py`，如下列程式碼中所述：

```cmd
python Experimentation.py [-h] -f FILE_PATH [-b BASE_COMMAND] [-p N_PROC]
                          [-s SHARED_NAMESPACES] [-a ACTION_NAMESPACES]
                          [-m MARGINAL_NAMESPACES] [--auto_lines AUTO_LINES]
                          [--only_hp] [-l LR_MIN_MAX_STEPS]
                          [-r REG_MIN_MAX_STEPS] [-t PT_MIN_MAX_STEPS]
                          [--q_bruteforce_terms Q_BRUTEFORCE_TERMS]
                          [--q_greedy_stop Q_GREEDY_STOP]
```

結果的記錄會附加至 *mwt-ds/DataScience/experiments.csv* 檔案。

### <a name="parameters"></a>參數
| 輸入 | 說明 | 預設值 |
| --- | --- | --- |
| `-h`、`--help` | 顯示說明訊息並結束。 | |
| `-f FILE_PATH`、`--file_path FILE_PATH` | 資料檔案路徑 (`.json` 或 `.json.gz` 格式 - 每行都是 `dsjson`)。 | 必要 |  
| `-b BASE_COMMAND`、`--base_command BASE_COMMAND` | 基本 Vowpal Wabbit 命令。  | `vw --cb_adf --dsjson -c` |  
| `-p N_PROC`、`--n_proc N_PROC` | 要使用的平行處理序數目。 | 邏輯處理器 |  
| `-s SHARED_NAMESPACES, --shared_namespaces SHARED_NAMESPACES` | 共用功能命名空間 (例如 `abc` 表示命名空間 `a`、`b` 和 `c`)。  | 從資料檔案自動偵測 |  
| `-a ACTION_NAMESPACES, --action_namespaces ACTION_NAMESPACES` | 動作功能命名空間。 | 從資料檔案自動偵測 |  
| `-m MARGINAL_NAMESPACES, --marginal_namespaces MARGINAL_NAMESPACES` | 臨界功能命名空間。 | 從資料檔案自動偵測 |  
| `--auto_lines AUTO_LINES` | 要掃描以自動偵測功能命名空間的資料檔案行數。 | `100` |  
| `--only_hp` | 只要對超參數 (`learning rate`、`L1 regularization` 和 `power_t`) 掃掠。 | `False` |  
| `-l LR_MIN_MAX_STEPS`、`--lr_min_max_steps LR_MIN_MAX_STEPS` | 學習速度範圍，以正值 `min,max,steps` 表示。 | `1e-5,0.5,4` |  
| `-r REG_MIN_MAX_STEPS`、`--reg_min_max_steps REG_MIN_MAX_STEPS` | L1 正規化範圍，以正值 `min,max,steps` 表示。 | `1e-9,0.1,5` |  
| `-t PT_MIN_MAX_STEPS`、`--pt_min_max_steps PT_MIN_MAX_STEPS` | Power_t 範圍，以正值 `min,max,step` 表示。 | `1e-9,0.5,5` |  
| `--q_bruteforce_terms Q_BRUTEFORCE_TERMS` | 要在暴力密碼破解階段中測試的二次方配對數。 | `2` |  
| `--q_greedy_stop Q_GREEDY_STOP` | 沒有改善的回合數，之後會停止二次方窮盡搜尋階段。 | `3` |  

### <a name="examples"></a>範例
若要使用預設值：
```cmd
python Experimentation.py -f D:\multiworld\data.json
```

同樣地，Vowpal Wabbit 也可以內嵌 `.json.gz` 檔案：
```cmd
python Experimentation.py -f D:\multiworld\data.json.gz
```

若只要對超參數 (`learning rate`、`L1 regularization` 和 `power_t`，步驟 2 後停止) 掃掠：
```cmd
python Experimentation.py -f D:\multiworld\data.json --only_hp
```
