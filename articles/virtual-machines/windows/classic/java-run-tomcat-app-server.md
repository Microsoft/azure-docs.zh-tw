---
title: 在傳統 Azure VM 上執行 Java 應用程式伺服器
description: 本教學課程使用以傳統部署模型建立的資源，示範如何建立 Windows 虛擬機器，並將其設定為執行 Apache Tomcat 應用程式伺服器。
services: virtual-machines-windows
documentationcenter: java
author: rmcmurray
manager: mbaldwin
tags: azure-service-management
ms.assetid: d627aa09-f7d6-4239-8110-f8fc5111b939
ms.service: virtual-machines-windows
ms.workload: web
ms.tgt_pltfrm: vm-windows
ms.devlang: Java
ms.topic: article
ms.date: 04/11/2018
ms.author: robmcm
ms.openlocfilehash: e13228a707e7dae4a4c2505154d01215c40b4716
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31528023"
---
# <a name="how-to-run-a-java-application-server-on-a-virtual-machine-created-with-the-classic-deployment-model"></a>如何在以傳統部署模型建立的虛擬機器上執行 Java 應用程式伺服器
> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如需 Resource Manager 範本使用 Java 8 與 Tomcat 部署 webapp 的詳細資訊，請參閱[這裡](https://azure.microsoft.com/documentation/templates/201-web-app-java-tomcat/)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

Azure 可讓您利用虛擬機器來提供伺服器功能。 例如，於 Azure 上執行的虛擬機器在經過設定後可代管 Apache Tomcat 等 Java 應用程式伺服器。

完成本指南之後，您將了解如何建立於 Azure 上執行的虛擬機器，以及如何設定才能使其執行 Java 應用程式伺服器。 您將了解與執行下列工作：

* 如何建立已安裝 Java Development Kit (JDK) 的虛擬機器。
* 如何從遠端登入虛擬機器。
* 如何將 Java 應用程式伺服器 (Apache Tomcat) 安裝在虛擬機器上。
* 如何為虛擬機器建立端點。
* 如何在防火牆中為應用程式伺服器開啟連接埠。

已完成的安裝會導致 Tomcat 在虛擬機器上執行。

![執行 Apache Tomcat 的虛擬機器][virtual_machine_tomcat]

[!INCLUDE [create-account-and-vms-note](../../../../includes/create-account-and-vms-note.md)]

## <a name="to-create-a-virtual-machine"></a>建立虛擬機器
1. 登入 [Azure 入口網站](https://portal.azure.com)。  
2. 依序按一下 [建立資源] 和 [計算]，然後按一下 [精選 App] 中的 [查看全部]。
3. 按一下 [JDK]，然後按一下 [JDK] 窗格中的 [JDK 8]。  
   如果您的舊版應用程式尚未準備好在 JDK 8 中運作，則可使用支援 **JDK 6** 和 **JDK 7** 的虛擬機器映像。
4. 在 [JDK 8] 窗格中，選取 [傳統]，然後按一下 [建立]。
5. 在 [基本] 刀鋒視窗中：
   1. 指定虛擬機器的名稱。
   2. 在 [使用者名稱]  欄位中輸入系統管理員的名稱。 請記住這個名稱以及下一個欄位中緊接著的相關聯密碼。 當您從遠端登入虛擬機器，需要用到這些資訊。
   3. 在 [新增密碼] 欄位中輸入密碼，然後在 [確認密碼] 欄位中再輸入一次。 此密碼適用於系統管理員帳戶。
   4. 選取適當的**訂用帳戶**。
   5. 針對**資源群組**，按一下 [建立新項目]，然後輸入新資源群組的名稱。 或者，按一下 [使用現有的項目]，然後選取其中一個可用的資源群組。
   6. 選取虛擬機器所在的位置，例如**美國中南部**。
6. 按 [下一步] 。
7. 在 [虛擬機器映像大小] 刀鋒視窗中，選取 [A1 標準] 或另一個適當的映像。
8. 按一下 [選取] 。

9. 在 [設定] 刀鋒視窗中，按一下 [確定]。 您可以使用 Azure 所提供的預設值。  
10. 在 [摘要] 刀鋒視窗中，按一下 [確定]。

## <a name="to-remotely-sign-in-to-your-virtual-machine"></a>從遠端登入虛擬機器
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [虛擬機器 (傳統)]。 您可以視需要按一下左下角位於服務類別下方的 [更多服務]。 [虛擬機器 (傳統)] 項目會列於 [計算] 群組中。
3. 按一下要登入的虛擬機器名稱。
4. 在啟動虛擬機器後，窗格頂端的功能表可允許連接。
5. 按一下 [ **連接**]。
6. 視需要回應提示以連接虛擬機器。 通常，您會儲存或開啟包含連接詳細資料的 .rdp 檔案。 您可能必須將 url:port 複製為 .rdp 檔案中第一行的最後一個部分，並將其貼入遠端登入應用程式中。

## <a name="to-install-a-java-application-server-on-your-virtual-machine"></a>將 Java 應用程式伺服器安裝在虛擬機器上
您可以將 Java 應用程式伺服器複製到虛擬機器，或您也可以透過安裝程式安裝 Java 應用程式伺服器。

本教學課程使用 Tomcat 做為要安裝的 Java 應用程式伺服器。

1. 登入虛擬機器後，開啟瀏覽器工作階段並前往 [Apache Tomcat](http://tomcat.apache.org/download-80.cgi)。
2. 按兩下 [32-bit/64-bit Windows Service Installer] 的連結。 利用此技術，Tomcat 將以 Windows 服務形式進行安裝。
3. 出現提示時，選擇執行安裝程式。
4. 在 [Apache Tomcat Setup]  精靈中，按照提示安裝 Tomcat。 基於本教學課程的目的，接受預設值即可。 待 [完成 Apache Tomcat 設定精靈] 對話方塊出現時，您可以選擇性勾選 [執行 Apache Tomcat]，以立即啟動 Tomcat。 按一下 [完成]  ，完成 Tomcat 安裝程序。

## <a name="to-start-tomcat"></a>啟動 Tomcat

您可以在虛擬機器上開啟命令提示字元，然後執行命令 **net&nbsp;start&nbsp;Tomcat8**，以手動方式啟動 Tomcat。

當 Tomcat 正在執行時，您可以在虛擬機器的瀏覽器中輸入 URL <http://localhost:8080>。

若要從外部機器看見執行中的 Tomcat，您需要建立端點並開啟連接埠。

## <a name="to-create-an-endpoint-for-your-virtual-machine"></a>為虛擬機器建立端點
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [虛擬機器 (傳統)]。
3. 按一下執行 Java 應用程式伺服器之虛擬機器的名稱。
4. 按一下 [端點] 。
5. 按一下 [新增] 。
6. 在 [新增端點] 對話方塊中：
   1. 指定端點的名稱，如 **HttpIn**。
   2. 針對通訊協定，選取 [TCP]。
   3. 指定公用連接埠 [80]  。
   4. 指定私人連接埠 [8080]  。
   5. 針對浮動 IP 位址，選取 [已停用]。
   6. 讓存取控制清單保留原狀。
   7. 按一下 [確定] 按鈕以關閉對話方塊，並建立端點。

## <a name="to-open-a-port-in-the-firewall-for-your-virtual-machine"></a>在防火牆中為虛擬機器開啟連接埠
1. 登入虛擬機器。
2. 按一下 Windows [開始] 。
3. 按一下 [控制台] 。
4. 依序按一下 [系統及安全性]、[Windows 防火牆] 及 [進階設定]。
5. 按一下 [輸入規則]，然後按一下 [新增規則]。  
   ![新增輸入規則][NewIBRule]
6. 針對 [規則類型]，選取 [連接埠]，然後按 [下一步]。  
   ![新增輸入規則連接埠][NewRulePort]
7. 在 [通訊協定和連接埠] 畫面上，選取 [TCP]，在 [特定本機連接埠] 中指定 **8080**，然後按 [下一步]。  
  ![新增輸入規則][NewRuleProtocol]
8. 在 [動作] 畫面上，選取 [允許連接]，然後按 [下一步]。
   ![新增輸入規則動作][NewRuleAction]
9. 在 [設定檔] 畫面上，確定已選取 [網域]、[私人] 和 [公用]，然後按 [下一步]。
   ![新增輸入規則設定檔][NewRuleProfile]
10. 在 [名稱] 畫面上，指定規則的名稱，例如 **HttpIn** (不過規則名稱不一定要與端點名稱相符)，然後按一下 [完成]。  
    ![新增輸入規則名稱][NewRuleName]

此時，應該能從外部瀏覽器檢視您的 Tomcat 網站。 在瀏覽器的位址視窗中，輸入 **http://*your\_DNS\_name*.cloudapp.net** 格式的 URL，其中 ***your\_DNS\_name*** 是您在建立虛擬機器時指定的 DNS 名稱。

## <a name="application-lifecycle-considerations"></a>應用程式生命週期考量
* 您可以自行建立 Web 應用程式封存檔 (WAR)，然後再將其新增至 **webapps** 資料夾。 例如，建立基本的 Java Service Page (JSP) 動態 Web 專案，並將它匯出為 WAR 檔案。 接下來，將 WAR 複製到虛擬機器上的 Apache Tomcat **webapps** 資料夾中，然後在瀏覽器中加以執行。
* 依預設，當您安裝 Tomcat 服務時，系統會將其設定為手動啟動。 您可以使用 [服務] 嵌入式管理單元將其切換為自動啟動。 依序按一下 Windows [開始]、[系統管理工具] 及 [服務] 以啟動 [服務] 嵌入式管理單元。 按兩下 [Apache Tomcat] 服務，然後將 [啟動類型] 設為 [自動]。

    ![將服務設定為自動啟動][service_automatic_startup]

    使 Tomcat 自動啟動的優點在於，當虛擬機器重新開機時 (例如安裝需要重新開機的軟體更新後)，它就會開始執行。

## <a name="next-steps"></a>後續步驟
您可以了解其他您可能想要納入 Java 應用程式中的服務，例如 Azure 儲存體、服務匯流排及 SQL Database。 請檢視 [Java 開發人員中心](https://azure.microsoft.com/develop/java/)上所提供的資訊。

[virtual_machine_tomcat]:media/java-run-tomcat-app-server/WA_VirtualMachineRunningApacheTomcat.png

[service_automatic_startup]:media/java-run-tomcat-app-server/WA_TomcatServiceAutomaticStart.png









[NewIBRule]:media/java-run-tomcat-app-server/NewInboundRule.png
[NewRulePort]:media/java-run-tomcat-app-server/NewRulePort.png
[NewRuleProtocol]:media/java-run-tomcat-app-server/NewRuleProtocol.png
[NewRuleAction]:media/java-run-tomcat-app-server/NewRuleAction.png
[NewRuleName]:media/java-run-tomcat-app-server/NewRuleName.png
[NewRuleProfile]:media/java-run-tomcat-app-server/NewRuleProfile.png


<!-- Deleted from the "To create an ednpoint for your virtual mache" 3/17/2017,
     to use the new portal.
6. In the **Add endpoint** dialog box, ensure **Add standalone endpoint** is selected, and then click **Next**.
7. In the **New endpoint details** dialog box:
-->
