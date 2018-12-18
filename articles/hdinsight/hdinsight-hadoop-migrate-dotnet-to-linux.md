---
title: 在以 Linux 為基礎的 HDInsight - Azure 上搭配 Hadoop MapReduce 使用 .NET
description: 了解如何在以 Linux 為基礎的 HDInsight 上使用 .NET 應用程式串流處理 MapReduce。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: f8a29c744d0ebfbab4127c421d0dba22e8d7ff52
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43111086"
---
# <a name="migrate-net-solutions-for-windows-based-hdinsight-to-linux-based-hdinsight"></a>將以 Windows 為基礎的 HDInsight 適用的 .NET 方案移轉至以 Linux 為基礎的 HDInsight

以 Linux 為基礎的 HDInsight 叢集會使用 [Mono (https://mono-project.com)](https://mono-project.com) 來執行 .NET 應用程式。 Mono 可讓您搭配以 Linux 為基礎的 HDInsight 使用 .NET 元件 (例如 MapReduce 應用程式)。 在本文件中，了解如何移轉為以 Windows 為基礎的 HDInsight 建立的 .NET 方案，以在以 Linux 為基礎的 HDInsight 上搭配 Mono 使用。

## <a name="mono-compatibility-with-net"></a>Mono 與 .NET 的相容性

4.2.1 版的 Mono 隨附於 3.6 版的 HDInsight。 如需 HDInsight 包含之 Mono 版本的詳細資訊，請參閱 [HDInsight 元件版本](hdinsight-component-versioning.md)。 若要安裝特定版本的 Mono，請參閱[安裝或更新 Mono](hdinsight-hadoop-install-mono.md) 文件。

如需 Mono 與 .NET 之間的相容性詳細資訊，請參閱 [Mono 相容性 (英文) (http://www.mono-project.com/docs/about-mono/compatibility/)](http://www.mono-project.com/docs/about-mono/compatibility/) 文件。

> [!IMPORTANT]
> SCP.NET 架構與 Mono 相容。 如需搭配 Mono 使用 SCP.NET 的詳細資訊，請參閱[使用 Visual Studio 開發適用於 Apache Storm on HDInsight 的 C# 拓撲](storm/apache-storm-develop-csharp-visual-studio-topology.md)。

## <a name="automated-portability-analysis"></a>自動化的可攜性分析

[.NET Portability Analyzer](https://marketplace.visualstudio.com/items?itemName=ConnieYau.NETPortabilityAnalyzer) 可用來產生一份您的應用程式和 Mono 之間的不相容性報告。 使用下列步驟設定分析器，以檢查您的應用程式是否具備 Mono 可攜性︰

1. 安裝 [.NET Portability Analyzer (英文)](https://marketplace.visualstudio.com/items?itemName=ConnieYau.NETPortabilityAnalyzer)。 在安裝期間，選取要使用的 Visual Studio 版本。

2. 從 Visual Studio 2015，選取 [分析] >  [Portability Analyzer 設定]，並確定已選取 [Mono] 區段中的 __4.5__。

    ![分析器設定的 [Mono] 區段中已選取 4.5](./media/hdinsight-hadoop-migrate-dotnet-to-linux/portability-analyzer-settings.png)

    選取 [確定] 以儲存組態。

3. 選取 [分析] > [分析組件可攜性]。 選取包含您的方案的組件，然後選取 [開啟] 以開始分析。

4. 分析完成之後，請選取 [分析] > [檢視分析報告]。 在 [可攜性分析結果] 中，選取 [開啟報告] 以開啟報告。

    ![Portability Analyzer 結果對話方塊](./media/hdinsight-hadoop-migrate-dotnet-to-linux/portability-analyzer-results.png)

> [!IMPORTANT]
> 分析器無法找出您方案的每個問題。 例如，如果 Mono 是在 Windows 上執行，`c:\temp\file.txt`的檔案路徑就會被視為 [確定]。 相同路徑在 Linux 平台上無效。

## <a name="manual-portability-analysis"></a>手動的可攜性分析

使用[應用程式可攜性 (英文) (http://www.mono-project.com/docs/getting-started/application-portability/)](http://www.mono-project.com/docs/getting-started/application-portability/) 文件中的資訊，執行程式碼的手動稽核。

## <a name="modify-and-build"></a>修改和建置

您可以繼續使用 Visual Studio 建置適用於 HDInsight 的 .NET 方案。 不過，您必須確定專案已設定為使用 .NET Framework 4.5。

## <a name="deploy-and-test"></a>部署和測試

使用 .NET Portability Analyzer 或手動分析的建議修改您的方案後，您必須使用 HDInsight 進行測試。 在以 Linux 為基礎的 HDInsight 叢集上測試方案，可能會揭露需要更正的細微問題。 建議您在測試時啟用應用程式中的其他記錄。

如需存取記錄的詳細資訊，請參閱下列文件：

* [存取以 Linux 為基礎之 HDInsight 上的 YARN 應用程式記錄](hdinsight-hadoop-access-yarn-app-logs-linux.md)

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 上搭配 MapReduce 使用 C#](hadoop/apache-hadoop-dotnet-csharp-mapreduce-streaming.md)

* [搭配 Hive 和 Pig 使用 C# 使用者定義函式](hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

* [開發適用於 Storm on HDInsight 的 C# 拓撲](storm/apache-storm-develop-csharp-visual-studio-topology.md)
