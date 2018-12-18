---
title: VM 上計算密集型 Java 應用程式
description: 了解如何建立可執行大量運算 Java 應用程式 (此應用程式又可由另一個 Java 應用程式監控) 的 Azure 虛擬機器。
services: virtual-machines-windows
documentationcenter: java
author: rmcmurray
manager: mbaldwin
tags: azure-service-management,azure-resource-manager
ms.assetid: ae6f2737-94c7-4569-9913-d871450c2827
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: Java
ms.topic: article
ms.date: 04/11/2018
ms.author: robmcm
ms.openlocfilehash: e8da296c30f1d2be3c637e456e90d2f93da67548
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31528455"
---
# <a name="how-to-run-a-compute-intensive-task-in-java-on-a-virtual-machine"></a>如何在虛擬機器上以 Java 執行大量運算工作
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

Azure 可讓您利用虛擬機器處理大量運算工作。 例如，虛擬機器可以處理工作並將結果傳遞給用戶端機器或行動裝置應用程式。 閱讀此文章後，您將了解如何建立一個執行大量運算之 Java 應用程式 (此應用程式又可由另一個 Java 應用程式監視) 的虛擬機器。

本教學課程是假設您知道如何建立 Java 主控台應用程式、將程式庫匯入您的 Java 應用程式，以及產生 Java 封存 (JAR)。 並假設您對 Microsoft Azure 一無所知。

您將了解：

* 如何建立已安裝 Java Development Kit (JDK) 的虛擬機器。
* 如何從遠端登入虛擬機器。
* 如何建立服務匯流排命名空間。
* 如何建立執行大量運算工作的 Java 應用程式。
* 如何建立監視大量運算工作進度的 Java 應用程式。
* 如何執行 Java 應用程式。
* 如何停止 Java 應用程式。

本教學課程將使用 Traveling Salesman Problem 進行需密集運算的工作。 以下是執行大量運算工作的 Java 應用程式範例。

![Traveling Salesman Problem solver][solver_output]

以下是監視大量運算工作的 Java 應用程式範例。

![Traveling Salesman Problem client][client_output]

[!INCLUDE [create-account-and-vms-note](../../../../includes/create-account-and-vms-note.md)]

## <a name="to-create-a-virtual-machine"></a>建立虛擬機器
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 依序按一下 [建立資源]、[計算]、[虛擬機器]，然後按一下 [從資源庫]。
3. 在 [虛擬機器映像選取] 對話方塊中，選取 [JDK 7 Windows Server 2012]。
   請注意，唯有當您擁有尚未做好在 JDK 7 中運作之準備的舊版應用程式時，才能選取 [JDK 6 Windows Server 2012]  。
4. 按 [下一步] 。
5. 在 [虛擬機器組態]  對話方塊中：
   1. 指定虛擬機器的名稱。
   2. 指定要用於虛擬機器的大小。
   3. 在 [使用者名稱]  欄位中輸入系統管理員的名稱。 請記住即將輸入的名稱和密碼，因為當您從遠端登入此虛擬機器時將需要用到它們。
   4. 在 [新增密碼] 欄位中輸入密碼，然後在 [確認] 欄位中再輸入一次。 此為系統管理員帳戶的密碼。
   5. 按 [下一步] 。
6. 在下一個 [虛擬機器組態]  對話方塊中：
   1. 針對 [雲端服務]，使用預設值 [建立新的雲端服務]。
   2. [Cloud service DNS name]  的值在整個 cloudapp.net 中必須是唯一的。 必要時請修改此值，使 Azure 指出該值是唯一的。
   3. 指定區域、同質群組或虛擬網路。 為因應本教學課程的目的，請指定如 [美國西部] 的區域。
   4. 針對 [儲存體帳戶]，選取 [使用自動產生的儲存體帳戶]。
   5. 針對 [可用性設定組]，選取 [(無)]。
   6. 按 [下一步] 。
7. 在最終的 [虛擬機器組態]  對話方塊中：
   1. 接受預設的端點項目。
   2. 按一下 [完成] 。

## <a name="to-remotely-log-in-to-your-virtual-machine"></a>從遠端登入虛擬機器
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [虛擬機器] 。
3. 按一下要登入的虛擬機器名稱。
4. 按一下 [ **連接**]。
5. 視需要回應提示以連接虛擬機器。 當要求提供系統管理員名稱和密碼的提示出現時，請使用在建立虛擬機器時提供的值。

請注意，Azure 服務匯流排功能會要求安裝 Baltimore CyberTrust Root 憑證做為您 JRE 的 **cacerts** 存放區的一部分。 這個憑證會自動包含在本教學課程所使用的 Java Runtime Environment (JRE) 中。 如果您的 JRE **cacerts** 存放區中沒有這個憑證，請參閱[將憑證新增至 Java CA 憑證存放區][add_ca_cert]，以了解有關新增這個憑證的資訊 (以及有關檢視您 cacerts 存放區中憑證的資訊)。

## <a name="how-to-create-a-service-bus-namespace"></a>如何建立服務匯流排命名空間
若要開始在 Azure 中使用服務匯流排佇列，首先必須建立服務命名空間。 服務命名空間提供範圍容器，可在應用程式內定址服務匯流排資源。

建立服務命名空間：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在 Azure 入口網站左下方的瀏覽窗格中，按一下 [服務匯流排、存取控制和快取]。
3. 在 Azure 入口網站的左上側窗格中，按一下 [服務匯流排] 節點，然後按一下 [新增] 按鈕。  
   ![Service Bus Node screenshot][svc_bus_node]
4. 在 [建立新的服務命名空間] 對話方塊中輸入一個**命名空間**，然後確認它是唯一的，再按一下 [檢查可用性] 按鈕。  
   ![Create a New Namespace screenshot][create_namespace]
5. 確定命名空間名稱可用之後，選擇要裝載命名空間的國家或地區，然後按一下 [建立命名空間]  按鈕。  
   
   然後，您建立的命名空間就會出現在 Azure 入口網站中，稍待片刻就會生效。 等到狀態變成 [作用中]  之後，再繼續下一個步驟。

## <a name="obtain-the-default-management-credentials-for-the-namespace"></a>取得命名空間的預設管理認證
若要在新的命名空間上執行管理作業，例如建立佇列，您必須取得命名空間的管理認證。

1. 在左方瀏覽窗格中，按一下 [服務匯流排]  節點，以顯示可用的命名空間清單。
   ![Available Namespaces screenshot][avail_namespaces]
2. 從顯示的清單中，選取您剛建立的命名空間。
   ![命名空間清單螢幕擷取畫面][namespace_list]
3. 右邊的 [屬性]  窗格會列出新命名空間的屬性。
   ![Properties Pane screenshot][properties_pane]
4. [預設金鑰]  是隱藏的。 請按一下 [檢視]  按鈕以顯示安全性認證。
   ![Default Key screenshot][default_key]
5. 記下 [預設核發者] 和 [預設金鑰]，因為您將在下面使用這項資訊來執行命名空間作業。

## <a name="how-to-create-a-java-application-that-performs-a-compute-intensive-task"></a>如何建立執行大量運算工作的 Java 應用程式
1. 在您的開發電腦 (不一定要是您所建立的虛擬機器) 上，下載 [Azure SDK for Java](https://azure.microsoft.com/develop/java/)(英文)。
2. 使用本節結尾的範例程式碼建立一個 Java 主控台應用程式。 在本教學課程中，我們將使用 **TSPSolver.java** 做為 Java 檔案名稱。 請將 **your\_service\_bus\_namespace**、**your\_service\_bus\_owner** 和 **your\_service\_bus\_key** 預留位置，分別修改成使用您服務匯流排 [命名空間]、[預設核發者] 及 [預設金鑰] 的值。
3. 編碼完成之後，將應用程式匯出至可執行的 Java 存檔 (JAR)，並將所需的程式庫封裝至產生的 JAR 中。 在本教學課程中，我們將使用 **TSPSolver.jar** 做為產生的 JAR 名稱。

<p/>

    // TSPSolver.java

    import com.microsoft.windowsazure.services.core.Configuration;
    import com.microsoft.windowsazure.services.core.ServiceException;
    import com.microsoft.windowsazure.services.serviceBus.*;
    import com.microsoft.windowsazure.services.serviceBus.models.*;
    import java.io.*;
    import java.text.DateFormat;
    import java.text.SimpleDateFormat;
    import java.util.ArrayList;
    import java.util.Date;
    import java.util.List;

    public class TSPSolver {

        //  Value specifying how often to provide an update to the console.
        private static long loopCheck = 100000000;  

        private static long nTimes = 0, nLoops=0;

        private static double[][] distances;
        private static String[] cityNames;
        private static int[] bestOrder;
        private static double minDistance;
        private static ServiceBusContract service;

        private static void buildDistances(String fileLocation, int numCities) throws Exception{
            try{
                BufferedReader file = new BufferedReader(new InputStreamReader(new DataInputStream(new FileInputStream(new File(fileLocation)))));
                double[][] cityLocs = new double[numCities][2];
                for (int i = 0; i<numCities; i++){
                    String[] line = file.readLine().split(", ");
                    cityNames[i] = line[0];
                    cityLocs[i][0] = Double.parseDouble(line[1]);
                    cityLocs[i][1] = Double.parseDouble(line[2]);
                }
                for (int i = 0; i<numCities; i++){
                    for (int j = i; j<numCities; j++){
                        distances[i][j] = Math.hypot(Math.abs(cityLocs[i][0] - cityLocs[j][0]), Math.abs(cityLocs[i][1] - cityLocs[j][1]));
                        distances[j][i] = distances[i][j];
                    }
                }
            } catch (Exception e){
                throw e;
            }
        }

        private static void permutation(List<Integer> startCities, double distSoFar, List<Integer> restCities) throws Exception {

            try
            {
                nTimes++;
                if (nTimes == loopCheck)
                {
                    nLoops++;
                    nTimes = 0;
                    DateFormat dateFormat = new SimpleDateFormat("MM/dd/yyyy HH:mm:ss");
                    Date date = new Date();
                    System.out.print("Current time is " + dateFormat.format(date) + ". ");
                    System.out.println(  "Completed " + nLoops + " iterations of size of " + loopCheck + ".");
                }

                if ((restCities.size() == 1) && ((minDistance == -1) || (distSoFar + distances[restCities.get(0)][startCities.get(0)] + distances[restCities.get(0)][startCities.get(startCities.size()-1)] < minDistance))){
                    startCities.add(restCities.get(0));
                    newBestDistance(startCities, distSoFar + distances[restCities.get(0)][startCities.get(0)] + distances[restCities.get(0)][startCities.get(startCities.size()-2)]);
                    startCities.remove(startCities.size()-1);
                }
                else{
                    for (int i=0; i<restCities.size(); i++){
                        startCities.add(restCities.get(0));
                        restCities.remove(0);
                        permutation(startCities, distSoFar + distances[startCities.get(startCities.size()-1)][startCities.get(startCities.size()-2)],restCities);
                        restCities.add(startCities.get(startCities.size()-1));
                        startCities.remove(startCities.size()-1);
                    }
                }
            }
            catch (Exception e)
            {
                throw e;
            }
        }

        private static void newBestDistance(List<Integer> cities, double distance) throws ServiceException, Exception {
            try
            {
                minDistance = distance;
                String cityList = "Shortest distance is "+minDistance+", with route: ";
                for (int i = 0; i<bestOrder.length; i++){
                    bestOrder[i] = cities.get(i);
                    cityList += cityNames[bestOrder[i]];
                    if (i != bestOrder.length -1)
                        cityList += ", ";
                }
                System.out.println(cityList);
                service.sendQueueMessage("TSPQueue", new BrokeredMessage(cityList));
            }
            catch (ServiceException se)
            {
                throw se;
            }
            catch (Exception e)
            {
                throw e;
            }
        }

        public static void main(String args[]){

            try {

                Configuration config = ServiceBusConfiguration.configureWithWrapAuthentication(
                        "your_service_bus_namespace", "your_service_bus_owner",
                        "your_service_bus_key",
                        ".servicebus.windows.net",
                        "-sb.accesscontrol.windows.net/WRAPv0.9");

                service = ServiceBusService.create(config);

                int numCities = 10;  // Use as the default, if no value is specified at command line.
                if (args.length != 0)
                {
                    if (args[0].toLowerCase().compareTo("createqueue")==0)
                    {
                        // No processing to occur other than creating the queue.
                        QueueInfo queueInfo = new QueueInfo("TSPQueue");

                        service.createQueue(queueInfo);

                        System.out.println("Queue named TSPQueue was created.");

                        System.exit(0);
                    }

                    if (args[0].toLowerCase().compareTo("deletequeue")==0)
                    {
                        // No processing to occur other than deleting the queue.
                        service.deleteQueue("TSPQueue");

                        System.out.println("Queue named TSPQueue was deleted.");

                        System.exit(0);
                    }

                    // Neither creating or deleting a queue.
                    // Assume the value passed in is the number of cities to solve.
                    numCities = Integer.valueOf(args[0]);  
                }

                System.out.println("Running for " + numCities + " cities.");

                List<Integer> startCities = new ArrayList<Integer>();
                List<Integer> restCities = new ArrayList<Integer>();
                startCities.add(0);
                for(int i = 1; i<numCities; i++)
                    restCities.add(i);
                distances = new double[numCities][numCities];
                cityNames = new String[numCities];
                buildDistances("c:\\TSP\\cities.txt", numCities);
                minDistance = -1;
                bestOrder = new int[numCities];
                permutation(startCities, 0, restCities);
                System.out.println("Final solution found!");
                service.sendQueueMessage("TSPQueue", new BrokeredMessage("Complete"));
            }
            catch (ServiceException se)
            {
                System.out.println(se.getMessage());
                se.printStackTrace();
                System.exit(-1);
            }
            catch (Exception e)
            {
                System.out.println(e.getMessage());
                e.printStackTrace();
                System.exit(-1);
            }
        }

    }



## <a name="how-to-create-a-java-application-that-monitors-the-progress-of-the-compute-intensive-task"></a>如何建立監視大量運算工作進度的 Java 應用程式
1. 在您的開發電腦上，使用本節結尾的範例程式碼建立一個 Java 主控台應用程式。 在本教學課程中，我們將使用 **TSPClient.java** 做為 Java 檔案名稱。 如上所示，請將 **your\_service\_bus\_namespace**、**your\_service\_bus\_owner** 和 **your\_service\_bus\_key** 預留位置，分別修改成使用您服務匯流排 [命名空間]、[預設核發者] 及 [預設金鑰] 的值。
2. 將應用程式匯出至可執行的 JAR，並將所需的程式庫封裝至產生的 JAR 中。 在本教學課程中，我們將使用 **TSPClient.jar** 做為產生的 JAR 名稱。

<p/>

    // TSPClient.java

    import java.util.Date;
    import java.text.DateFormat;
    import java.text.SimpleDateFormat;
    import com.microsoft.windowsazure.services.serviceBus.*;
    import com.microsoft.windowsazure.services.serviceBus.models.*;
    import com.microsoft.windowsazure.services.core.*;

    public class TSPClient
    {

        public static void main(String[] args)
        {
                try
                {

                    DateFormat dateFormat = new SimpleDateFormat("MM/dd/yyyy HH:mm:ss");
                    Date date = new Date();
                    System.out.println("Starting at " + dateFormat.format(date) + ".");

                    String namespace = "your_service_bus_namespace";
                    String issuer = "your_service_bus_owner";
                    String key = "your_service_bus_key";

                    Configuration config;
                    config = ServiceBusConfiguration.configureWithWrapAuthentication(
                            namespace, issuer, key,
                            ".servicebus.windows.net",
                            "-sb.accesscontrol.windows.net/WRAPv0.9");

                    ServiceBusContract service = ServiceBusService.create(config);

                    BrokeredMessage message;

                    int waitMinutes = 3;  // Use as the default, if no value is specified at command line.
                    if (args.length != 0)
                    {
                        waitMinutes = Integer.valueOf(args[0]);  
                    }

                    String waitString;

                    waitString = (waitMinutes == 1) ? "minute." : waitMinutes + " minutes.";

                    // This queue must have previously been created.
                    service.getQueue("TSPQueue");

                    int numRead;

                    String s = null;

                    while (true)
                    {

                        ReceiveQueueMessageResult resultQM = service.receiveQueueMessage("TSPQueue");
                        message = resultQM.getValue();

                        if (null != message && null != message.getMessageId())
                        {

                            // Display the queue message.
                            byte[] b = new byte[200];

                            System.out.print("From queue: ");

                            s = null;
                            numRead = message.getBody().read(b);
                            while (-1 != numRead)
                            {
                                s = new String(b);
                                s = s.trim();
                                System.out.print(s);
                                numRead = message.getBody().read(b);
                            }
                            System.out.println();
                            if (s.compareTo("Complete") == 0)
                            {
                                // No more processing to occur.
                                date = new Date();
                                System.out.println("Finished at " + dateFormat.format(date) + ".");
                                break;
                            }
                        }
                        else
                        {
                            // The queue is empty.
                            System.out.println("Queue is empty. Sleeping for another " + waitString);
                            Thread.sleep(60000 * waitMinutes);
                        }
                    }

            }
            catch (ServiceException se)
            {
                System.out.println(se.getMessage());
                se.printStackTrace();
                System.exit(-1);
            }
            catch (Exception e)
            {
                System.out.println(e.getMessage());
                e.printStackTrace();
                System.exit(-1);
            }

        }

    }

## <a name="how-to-run-the-java-applications"></a>如何執行 Java 應用程式
請執行大量運算應用程式，先建立佇列，然後解決旅行業務員問題，這會將目前的最佳路徑加入至服務匯流排佇列。 在大量運算應用程式執行時 (或執行後)，請執行用戶端以顯示來自服務匯流排佇列的結果。

### <a name="to-run-the-compute-intensive-application"></a>若要執行大量運算應用程式
1. 登入虛擬機器。
2. 建立將執行您應用程式的資料夾。 例如 **c:\TSP**。
3. 將 **TSPSolver.jar** 複製到 **c:\TSP**。
4. 建立一個含有下列內容且名為 **c:\TSP\cities.txt** 的檔案。
   
        City_1, 1002.81, -1841.35
        City_2, -953.55, -229.6
        City_3, -1363.11, -1027.72
        City_4, -1884.47, -1616.16
        City_5, 1603.08, -1030.03
        City_6, -1555.58, 218.58
        City_7, 578.8, -12.87
        City_8, 1350.76, 77.79
        City_9, 293.36, -1820.01
        City_10, 1883.14, 1637.28
        City_11, -1271.41, -1670.5
        City_12, 1475.99, 225.35
        City_13, 1250.78, 379.98
        City_14, 1305.77, 569.75
        City_15, 230.77, 231.58
        City_16, -822.63, -544.68
        City_17, -817.54, -81.92
        City_18, 303.99, -1823.43
        City_19, 239.95, 1007.91
        City_20, -1302.92, 150.39
        City_21, -116.11, 1933.01
        City_22, 382.64, 835.09
        City_23, -580.28, 1040.04
        City_24, 205.55, -264.23
        City_25, -238.81, -576.48
        City_26, -1722.9, -909.65
        City_27, 445.22, 1427.28
        City_28, 513.17, 1828.72
        City_29, 1750.68, -1668.1
        City_30, 1705.09, -309.35
        City_31, -167.34, 1003.76
        City_32, -1162.85, -1674.33
        City_33, 1490.32, 821.04
        City_34, 1208.32, 1523.3
        City_35, 18.04, 1857.11
        City_36, 1852.46, 1647.75
        City_37, -167.44, -336.39
        City_38, 115.4, 0.2
        City_39, -66.96, 917.73
        City_40, 915.96, 474.1
        City_41, 140.03, 725.22
        City_42, -1582.68, 1608.88
        City_43, -567.51, 1253.83
        City_44, 1956.36, 830.92
        City_45, -233.38, 909.93
        City_46, -1750.45, 1940.76
        City_47, 405.81, 421.84
        City_48, 363.68, 768.21
        City_49, -120.3, -463.13
        City_50, 588.51, 679.33
5. 在命令提示字元，將目錄切換至 c:\TSP。
6. 確定 JRE 的 bin 資料夾在 PATH 環境變數中。
7. 您將需要在執行 TSP 求解器排列之前，先建立服務匯流排佇列。 請執行下列命令來建立服務匯流排佇列。
   
        java -jar TSPSolver.jar createqueue
8. 建立完佇列之後，您便可以執行 TSP 求解器排列。 例如，執行下列命令來執行 8 個城市的求解器。
   
        java -jar TSPSolver.jar 8
   
   如果您沒有指定數目，它將會針對 10 個城市執行。 當 solver 找到目前最短路由時，便會將這些路由新增至佇列。

> [!NOTE]
> 您指定的數目越大，求解器就會花越長的時間執行。 例如，針對 14 個城市執行可能需花數分鐘，而針對 15 個城市執行可能需花數小時。 增加至 16 個或更多城市可能需花數天執行 (最終可能達數周、數月及數年)。 這是因為隨著城市數目增加，求解器所評估的排列數目也隨之激增的緣故。
> 
> 

### <a name="how-to-run-the-monitoring-client-application"></a>如何執行監視用戶端應用程式
1. 登入您將執行用戶端應用程式的電腦。 這不一定要是執行 **TSPSolver** 應用程式的同一部電腦，但也可以是同一部電腦。
2. 建立將執行您應用程式的資料夾。 例如 **c:\TSP**。
3. 將 **TSPClient.jar** 複製到 **c:\TSP**。
4. 確定 JRE 的 bin 資料夾在 PATH 環境變數中。
5. 在命令提示字元，將目錄切換至 c:\TSP。
6. 執行下列命令。
   
        java -jar TSPClient.jar
   
    您可以選擇是否要傳遞命令列引數來指定檢查佇列之間的睡眠分鐘數。 檢查佇列的預設睡眠期間為 3 分鐘，若未將任何命令列引數傳遞至 **TSPClient**，便會使用此預設值。 如果您想要使用其他值 (例如 1 分鐘) 做為睡眠間隔，請執行下列命令。
   
        java -jar TSPClient.jar 1
   
    用戶端將會執行直到看見 [完成] 佇列訊息為止。 請注意，如果您在未執行用戶端的情況下執行多個求解器，您可能需要執行用戶端多次，才能完全清空佇列。 或者，您也可以刪除佇列，然後重新建立它。 若要刪除此佇列，請執行下列 **TSPSolver** (而非 **TSPClient**) 命令。
   
        java -jar TSPSolver.jar deletequeue
   
    求解器將會執行直到完成所有路徑檢查為止。

## <a name="how-to-stop-the-java-applications"></a>如何停止 Java 應用程式
如果您想要在正常完成前結束 Solver 或用戶端應用程式，都可以按 **Ctrl+C** 來結束。

[solver_output]:media/java-run-compute-intensive-task/WA_JavaTSPSolver.png
[client_output]:media/java-run-compute-intensive-task/WA_JavaTSPClient.png
[svc_bus_node]:media/java-run-compute-intensive-task/SvcBusQueues_02_SvcBusNode.jpg
[create_namespace]:media/java-run-compute-intensive-task/SvcBusQueues_03_CreateNewSvcNamespace.jpg
[avail_namespaces]:media/java-run-compute-intensive-task/SvcBusQueues_04_SvcBusNode_AvailNamespaces.jpg
[namespace_list]:media/java-run-compute-intensive-task/SvcBusQueues_05_NamespaceList.jpg
[properties_pane]:media/java-run-compute-intensive-task/SvcBusQueues_06_PropertiesPane.jpg
[default_key]:media/java-run-compute-intensive-task/SvcBusQueues_07_DefaultKey.jpg
[add_ca_cert]: ../../../java-add-certificate-ca-store.md
