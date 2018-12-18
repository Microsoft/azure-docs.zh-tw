---
title: 了解使用 Hadoop 沙箱 - 模擬器 - Azure HDInsight
description: '若要開始了解 Hadoop 生態系統，您可以在 Azure 虛擬機器上從 Hortonworks 設定 Hadoop 沙箱。 '
keywords: hadoop 模擬器, hadoop 沙箱
ms.reviewer: jasonh
services: hdinsight
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 12/11/2017
ms.author: jasonh
ms.openlocfilehash: 66450077de4748bcd8703080d33f37169671ebe3
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048694"
---
# <a name="get-started-with-a-hadoop-sandbox-an-emulator-on-a-virtual-machine"></a>開始使用 Hadoop 沙箱，它是虛擬機器上的模擬器

了解如何在虛擬機器上從 Hortonworks 安裝 Hadoop 沙箱，以了解 Hadoop 生態系統。 沙箱提供本機開發環境，讓您了解 Hadoop、Hadoop 分散式檔案系統 (HDFS)，以及作業提交。 熟悉 Hadoop 之後，您就可以開始在 Azure 中使用 Hadoop 建立 HDInsight 叢集。 有關如何開始使用的詳細資訊，請參閱 [開始在 HDInsight 中使用 Hadoop](apache-hadoop-linux-tutorial-get-started.md)。

## <a name="prerequisites"></a>必要條件
* [Oracle VirtualBox](https://www.virtualbox.org/)。 請從[這裡](https://www.virtualbox.org/wiki/Downloads)下載並安裝。



## <a name="download-and-install-the-virtual-machine"></a>下載並安裝虛擬機器
1. 瀏覽至 [Hortonworks 下載](http://hortonworks.com/downloads/#sandbox)。

2. 按一下 [DOWNLOAD FOR VIRTUALBOX]\(適用於 VIRTUALBOX 的下載項目\)，以下載最新的 VM 上「Hortonworks 沙箱」。 系統會提示您向 Hortonworks 註冊，然後才能開始下載。 下載需要一到兩個小時的時間，視您的網路速度而定。
   
    ![用於下載 VirtualBox 之 Hortonworks 沙箱的連結影像](./media/apache-hadoop-emulator-get-started/download-sandbox.png)
3. 從相同網頁中，按一下 [Import on Virtual Box] \(在 Virtual Box 上匯入) 連結，以下載包含虛擬機器安裝指示的 PDF。

若要下載舊版 HDP 沙箱，請展開封存：

![Hortonworks 沙箱封存](./media/apache-hadoop-emulator-get-started/hortonworks-sandbox-archive.png)


## <a name="start-the-virtual-machine"></a>啟動虛擬機器

1. 開啟 [Oracle VM VirtualBox]。
2. 從 [檔案] 功能表中，按一下 [匯入設備]，然後指定「Hortonworks 沙箱」映像。
1. 選取「Hortonworks 沙箱」、按一下 [啟動]，然後選取 [正常啟動]。 虛擬機器完成開機程序後會顯示登入指示。
   
    ![正常啟動](./media/apache-hadoop-emulator-get-started/normal-start.png)
2. 開啟網頁瀏覽器，並瀏覽至顯示的 URL (通常是 http://127.0.0.1:8888) )。

## <a name="set-sandbox-passwords"></a>設定沙箱密碼

1. 從 [Hortonworks 沙箱] 頁面的 [入門] 步驟選取 [檢視進階選項]。 在此頁面上使用該資訊，以透過 SSH 登入沙箱。 使用提供的名稱和密碼。
   
   > [!NOTE]
   > 如果您未安裝 SSH 用戶端，您可以使用虛擬機器在 **http://localhost:4200/** 所提供的網頁型 SSH。
   > 
   
    第一次使用 SSH 連線時，系統會提示您變更根帳戶的密碼。 輸入新密碼，這是您在使用 SSH 登入時使用的密碼。

2. 登入之後，輸入下列命令：
   
        ambari-admin-password-reset
   
    出現提示時，提供 Ambari 系統管理員帳戶的密碼。 當您存取 Ambari Web UI 時將會用到。

## <a name="use-hive-commands"></a>使用 Hive 命令

1. 在連往沙箱的 SSH 連線中，使用下列命令啟動 Hive 殼層：
   
        hive
2. 殼層啟動之後，使用下列命令檢視沙箱所提供的資料表︰
   
        show tables;
3. 使用下列命令從 `sample_07` 資料表擷取 10 個資料列︰
   
        select * from sample_07 limit 10;

## <a name="next-steps"></a>後續步驟
* [了解如何搭配使用 Visual Studio 與 Hortonworks 沙箱](../hdinsight-hadoop-emulator-visual-studio.md)
* [了解 Hortonworks 沙箱的訣竅](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)
* [Hadoop 教學課程 - 開始使用 HDP](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/)

