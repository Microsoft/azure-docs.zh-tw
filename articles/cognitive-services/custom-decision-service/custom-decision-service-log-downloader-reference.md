---
title: LogDownloader - Azure 認知服務 | Microsoft Docs
description: 下載 Azure 自訂決策服務產生的記錄檔。
services: cognitive-services
author: marco-rossi29
manager: marco-rossi29
ms.service: cognitive-services
ms.topic: article
ms.date: 05/09/2018
ms.author: marossi
ms.openlocfilehash: 783b534b3b3f4bb7f5d9f073f491690759edfea5
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369955"
---
# <a name="logdownloader"></a>LogDownloader

下載 Azure 自訂決策檔案產生的記錄檔，並產生測試使用的 *.gz* 檔案。

## <a name="prerequisites"></a>先決條件

- Python 3：已安裝並在您的路徑上。 建議使用 64 位元版本以處理大型檔案。
- *Microsoft/mwt-ds* 存放庫：[複製存放庫](https://github.com/Microsoft/mwt-ds)。
- The *azure-storage-blob* 封裝：如需安裝詳細資料，請移至[適用於 Python 的 Microsoft Azure 儲存體程式庫](https://github.com/Azure/azure-storage-python#option-1-via-pypi)。
- 在 *mwt-ds/DataScience/ds.config* 中輸入您的 Azure 儲存體連接字串：遵循 *my_app_id: my_connectionString* 範本。 您可以指定多個 `app_id`。 當您執行 `LogDownloader.py` 時，如果 `ds.config` 中找不到輸入的 `app_id`，`LogDownloader.py` 會使用 `$Default` 連接字串。

## <a name="usage"></a>使用量

移至 `mwt-ds/DataScience` 並使用相關引數執行 `LogDownloader.py`，如下列程式碼中所述：

```cmd
python LogDownloader.py [-h] -a APP_ID -l LOG_DIR [-s START_DATE]
                        [-e END_DATE] [-o OVERWRITE_MODE] [--dry_run]
                        [--create_gzip] [--delta_mod_t DELTA_MOD_T]
                        [--verbose] [-v VERSION]
```

### <a name="parameters"></a>參數

| 輸入 | 說明 | 預設值 |
| --- | --- | --- |
| `-h`、`--help` | 顯示說明訊息並結束。 | |
| `-a APP_ID`、`--app_id APP_ID` | 應用程式識別碼 (也就是 Azure 儲存體 Blob 容器名稱)。 | 必要 |
| `-l LOG_DIR`、`--log_dir LOG_DIR` | 下載資料的基礎目錄 (已建立子資料夾)。  | 必要 |
| `-s START_DATE`、`--start_date START_DATE` | 下載開始日期 (包含)，格式為 *YYYY-MM-DD*。 | `None` |
| `-e END_DATE`、`--end_date END_DATE` | 下載結束日期 (包含)，格式為 *YYYY-MM-DD*。 | `None` |
| `-o OVERWRITE_MODE`、`--overwrite_mode OVERWRITE_MODE` | 可使用的覆寫模式。 | |
| | `0`：永不覆寫；詢問使用者目前是否使用 Blob。 | 預設值 | |
| | `1`：當檔案具有不同大小或目前正在使用 Blob 時，詢問使用者如何繼續。 | |
| | `2`：一律覆寫；下載目前使用的 Blob。 | |
| | `3`：永不覆寫，如果是大尺寸，則附加，且不詢問；下載目前使用的 Blob。 | |
| | `4`：永不覆寫，如果是大尺寸，則附加，且不詢問；跳過目前使用的 Blob。 | |
| `--dry_run` | 列印已下載的 Blob，而不需要下載。 | `False` |
| `--create_gzip` | 為 Vowpal Wabbit 建立 *gzip* 檔案。 | `False` |
| `--delta_mod_t DELTA_MOD_T` | 時間範圍，以秒為單位，用於偵測檔案是否正在使用中。 | `3600` 秒 (`1` 小時) |
| `--verbose` | 列印更多詳細資料。 | `False` |
| `-v VERSION`、`--version VERSION` | 要使用的記錄程式下載程式版本。 | |
| | `1`：用於未經處理的記錄 (僅適用於回溯相容性)。 | 取代 |
| | `2`：適用於未處理過的記錄。 | 預設值 |

### <a name="examples"></a>範例

如需在 Azure 儲存體 Blob 容器中試執行下載所有資料，請使用下列程式碼：
```cmd
python LogDownloader.py -a your_app_id -l d:\data --dry_run
```

若要透過 `overwrite_mode=4` 僅下載從 2018 年 1 月 1 日之後建立的記錄，請使用下列程式碼：
```cmd
python LogDownloader.py -a your_app_id -l d:\data -s 2018-1-1 -o 4
```

若要建立合併所有下載檔案的 *gzip* 檔案，請使用下列程式碼：
```cmd
python LogDownloader.py -a your_app_id -l d:\data -s 2018-1-1 -o 4 --create_gzip
```