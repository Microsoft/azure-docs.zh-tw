---
title: 使用 Azure Data Lake Storage Gen2 預覽版 URI
description: 使用 Azure Data Lake Storage Gen2 預覽版 URI
services: storage
keywords: ''
author: jamesbak
ms.topic: article
ms.author: jamesbak
ms.date: 06/27/2018
ms.service: storage
ms.component: data-lake-storage-gen2
ms.openlocfilehash: 71c3343db592162bcfa7ec46a29e0d680ca83079
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48900701"
---
# <a name="use-the-azure-data-lake-storage-gen2-uri"></a>使用 Azure Data Lake Storage Gen2 URI

與 Azure Data Lake Storage Gen2 預覽版相容的 [Hadoop Filesystem](http://www.aosabook.org/en/hdfs.html) 驅動程式，以其配置識別碼 `abfs` (Azure Blob 檔案系統) 而聞名。 ABFS 驅動程式和其他 Hadoop Filesystem 驅動程式一樣，都使用 URI 格式來處理 Data Lake Storage Gen2 支援帳戶中的檔案和目錄。

## <a name="uri-syntax"></a>URI 語法

Data Lake Storage Gen2 的 URI 語法取決於您的儲存體帳戶是否設定為將 Data Lake Storage Gen2 作為預設的檔案系統。

如果您希望處理的 Data Lake Storage Gen2 可使用帳戶**未**在帳戶建立期間設為預設的檔案系統，URI 語法速記為：

<pre>abfs[s]<sup>1</sup>://&lt;file_system&gt;<sup>2</sup>@&lt;account_name&gt;<sup>3</sup>.dfs.core.windows.net/&lt;path&gt;<sup>4</sup>/&lt;file_name&gt;<sup>5</sup></pre>

1. **配置識別碼**：此 `abfs` 通訊協定作為配置識別碼。 您可以選擇是否使用安全通訊端層 (SSL) 連線進行連線。 使用 `abfss` 可透過安全通訊端層連線進行連線。

2. **檔案系統**：保存檔案和資料夾的上層位置。 這與 Azure 儲存體 Blob 服務中的容器相同。

3. **帳戶名稱**：建立期間為儲存體帳戶指定的名稱。

4. **路徑**：正斜線分隔 (`/`) 的形式表示目錄結構。

5. **檔案名稱**：個別檔案的名稱。 如果要定址目錄，可選用此參數。

但是，如果您希望處理的帳戶在帳戶建立期間已設為預設的檔案系統，則 URI 語法速記為：

<pre>/&lt;path&gt;<sup>1</sup>/&lt;file_name&gt;<sup>2</sup></pre>

1. **路徑**：正斜線分隔 (`/`) 的形式表示目錄結構。

2. **檔案名稱**：個別檔案的名稱。


## <a name="next-steps"></a>後續步驟

- [搭配 Azure HDInsight 叢集使用 Data Lake Storage Gen2](use-hdi-cluster.md)
