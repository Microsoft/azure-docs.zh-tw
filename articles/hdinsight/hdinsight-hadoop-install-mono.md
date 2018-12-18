---
title: 在 HDInsight 上安裝或更新 Mono - Azure
description: 了解如何搭配 HDInsight 叢集使用特定版本的 Mono。 Mono 是用於在以 Linux 為基礎的 HDInsight 叢集上執行 .NET 應用程式。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/10/2018
ms.author: jasonh
ms.custom: hdinsightactive
ms.openlocfilehash: 54e5a5b72627dc9cd2d842ccb24d10e2f9ab9dce
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957171"
---
# <a name="install-or-update-mono-on-hdinsight"></a>在 HDInsight 上安裝或更新 Mono

了解如何在 HDInsight 3.4 或更新版本上安裝特定版本的 [Mono](https://www.mono-project.com)。

Mono 安裝在 HDInsight 3.4 和更新版本，並用來執行 .NET 應用程式。 如需每個 HDInsight 版本包含之 Mono 版本的相關資訊，請參閱 [HDInsight 元件版本設定](hdinsight-component-versioning.md)。 若要在叢集上安裝不同版本，請使用此文件中的指令碼動作。 

## <a name="how-it-works"></a>運作方式

此指令碼接受下列參數︰

* __Mono 版本號碼__：要安裝的 Mono 版本。 版本必須可從 [https://download.mono-project.com/repo/debian/dists/wheezy/snapshots/](https://download.mono-project.com/repo/debian/dists/wheezy/snapshots/) 取得。

指令碼會安裝下列 Mono 套件︰

* __mono-complete__

* __ca-certificates-mono__

## <a name="the-script"></a>指令碼

__指令碼位置__：[https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash](https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash)

__需求__：

* 指令碼必須套用於__前端節點__和__背景工作節點__。

## <a name="to-use-the-script"></a>使用指令碼

如需如何搭配 HDInsight 使用此指令碼的資訊，請參閱[使用指令碼動作來自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md#apply-a-script-action-to-a-running-cluster)文件。 您可以透過 Azure 入口網站、Azure PowerShell 或 Azure 傳統 CLI 來使用指令碼。

遵循指令碼動作文件時，請使用下列 URI：

    https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash

若要指定已安裝的 Mono 版本，請使用 [參數] 欄位中的版本號碼。 例如，輸入 `5.4` 可安裝 Mono 5.4。

> [!NOTE]
> 使用此指令碼設定 HDInsight 時，請將指令碼標示為 __Persisted__。 此設定可讓 HDInsight 將指令碼套用到透過調整作業所新增的背景工作節點。

## <a name="next-steps"></a>後續步驟

您已了解如何在 HDInsight 上進行升級或安裝特定版本的 Mono。 如需在 HDInsight 上搭配 Mono 使用 .NET 應用程式的詳細資訊，請參閱下列文件：

* [在 HDInsight 上使用 .NET 進行串流 MapReduce](hadoop/apache-hadoop-dotnet-csharp-mapreduce-streaming.md)
* [在 HDInsight 上搭配 Hive 與 Pig 使用 .NET](hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)
* [在 HDInsight 上使用 Storm 開發 C# 方案](storm/apache-storm-develop-csharp-visual-studio-topology.md)
* [將 .NET 方案移轉至以 Linux 為基礎的 HDInsight](hdinsight-hadoop-migrate-dotnet-to-linux.md)

如需使用指令碼動作的詳細資訊，請參閱[使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)
