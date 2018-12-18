---
title: 使用 JAVA 來建立和管理 Azure 虛擬機器 | Microsoft Docs
description: 使用 JAVA 和 Azure Resource Manager 來部署虛擬機器及其所有支援的資源。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.author: cynthn
ms.openlocfilehash: e4c77514bb38b6c3c5766bdb4af22e5271c9bffa
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31525225"
---
# <a name="create-and-manage-windows-vms-in-azure-using-java"></a>在 Azure 中使用 JAVA 建立並管理 Windows VM

[Azure 虛擬機器](overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (VM) 需要數個支援的 Azure 資源。 本文涵蓋使用 JAVA 建立、管理和刪除 VM 資源。 您會了解如何：

> [!div class="checklist"]
> * 建立 Maven 專案
> * 新增相依性
> * 建立認證
> * 建立資源
> * 執行管理工作
> * 刪除資源
> * 執行應用程式

執行這些步驟大約需要 20 分鐘的時間。

## <a name="create-a-maven-project"></a>建立 Maven 專案

1. 如果尚未這麼做，請先安裝 [JAVA](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。
2. 安裝 [Maven](http://maven.apache.org/download.cgi)。
3. 建立新的資料夾和專案：
    
    ```
    mkdir java-azure-test
    cd java-azure-test
    
    mvn archetype:generate -DgroupId=com.fabrikam -DartifactId=testAzureApp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

## <a name="add-dependencies"></a>新增相依性

1. 在 `testAzureApp` 資料夾下，開啟 `pom.xml` 檔案，然後將組建設定新增至&lt;專案&gt;，以啟用您的應用程式的建置：

    ```xml
    <build>
      <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <configuration>
                <mainClass>com.fabrikam.testAzureApp.App</mainClass>
            </configuration>
        </plugin>
      </plugins>
    </build>
    ```

2. 新增存取 Azure Java SDK 所需的相依性。

    ```xml
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-compute</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-resources</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-network</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.squareup.okio</groupId>
      <artifactId>okio</artifactId>
      <version>1.13.0</version>
    </dependency>
    <dependency> 
      <groupId>com.nimbusds</groupId>
      <artifactId>nimbus-jose-jwt</artifactId>
      <version>3.6</version>
    </dependency>
    <dependency>
      <groupId>net.minidev</groupId>
      <artifactId>json-smart</artifactId>
      <version>1.0.6.3</version>
    </dependency>
    <dependency>
      <groupId>javax.mail</groupId>
      <artifactId>mail</artifactId>
      <version>1.4.5</version>
    </dependency>
    ```

3. 儲存檔案。

## <a name="create-credentials"></a>建立認證

在您開始此步驟之前，請確定您可以存取 [Active Directory 服務主體](../../azure-resource-manager/resource-group-create-service-principal-portal.md)。 您也應該記錄應用程式識別碼、驗證金鑰以及租用戶識別碼，您在稍後的步驟會需要這些項目。

### <a name="create-the-authorization-file"></a>建立授權檔

1. 建立名為 `azureauth.properties` 的檔案名稱，並加入下列屬性：

    ```
    subscription=<subscription-id>
    client=<application-id>
    key=<authentication-key>
    tenant=<tenant-id>
    managementURI=https://management.core.windows.net/
    baseURL=https://management.azure.com/
    authURL=https://login.windows.net/
    graphURL=https://graph.windows.net/
    ```

    以您的訂用帳戶 ID 取代 **&lt;subscription-id&gt;**、以 Active Directory 應用程式識別碼取代 **&lt;application-id&gt;**、以應用程式金鑰取代 **&lt;authentication-key&gt;**，以及以租用戶識別碼取代 **&lt;tenant-id&gt;**。

2. 儲存檔案。
3. 使用驗證檔案的完整路徑，在殼層中設定名稱為 AZURE_AUTH_LOCATION 的環境變數。

### <a name="create-the-management-client"></a>建立管理用戶端

1. 在 `src\main\java\com\fabrikam` 下開啟 `App.java` 檔案，並確定此 package 陳述式位於最上方：

    ```java
    package com.fabrikam.testAzureApp;
    ```

2. 在 package 陳述式下，新增下列 import 陳述式：
   
    ```java
    import com.microsoft.azure.management.Azure;
    import com.microsoft.azure.management.compute.AvailabilitySet;
    import com.microsoft.azure.management.compute.AvailabilitySetSkuTypes;
    import com.microsoft.azure.management.compute.CachingTypes;
    import com.microsoft.azure.management.compute.InstanceViewStatus;
    import com.microsoft.azure.management.compute.DiskInstanceView;
    import com.microsoft.azure.management.compute.VirtualMachine;
    import com.microsoft.azure.management.compute.VirtualMachineSizeTypes;
    import com.microsoft.azure.management.network.PublicIPAddress;
    import com.microsoft.azure.management.network.Network;
    import com.microsoft.azure.management.network.NetworkInterface;
    import com.microsoft.azure.management.resources.ResourceGroup;
    import com.microsoft.azure.management.resources.fluentcore.arm.Region;
    import com.microsoft.azure.management.resources.fluentcore.model.Creatable;
    import com.microsoft.rest.LogLevel;
    import java.io.File;
    import java.util.Scanner;
    ```

2. 若要建立您提出要求所需的 Active Directory 認證，請將此程式碼新增至 App 類別的 Main 方法：
   
    ```java
    try {    
        final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));
        Azure azure = Azure.configure()
            .withLogLevel(LogLevel.BASIC)
            .authenticate(credFile)
            .withDefaultSubscription();
    } catch (Exception e) {
        System.out.println(e.getMessage());
        e.printStackTrace();
    }

    ```

## <a name="create-resources"></a>建立資源

### <a name="create-the-resource-group"></a>建立資源群組

所有資源都必須包含在[資源群組](../../azure-resource-manager/resource-group-overview.md)中。

若要指定應用程式的值並建立資源群組，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Creating resource group...");
ResourceGroup resourceGroup = azure.resourceGroups()
    .define("myResourceGroup")
    .withRegion(Region.US_EAST)
    .create();
```

### <a name="create-the-availability-set"></a>建立可用性設定組

維護您應用程式所使用的虛擬機器時，使用[可用性設定組](tutorial-availability-sets.md)可讓您更加輕鬆。

若要建立可用性設定組，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Creating availability set...");
AvailabilitySet availabilitySet = azure.availabilitySets()
    .define("myAvailabilitySet")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withSku(AvailabilitySetSkuTypes.MANAGED)
    .create();
```
### <a name="create-the-public-ip-address"></a>建立公用 IP 位址

必須要有[公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)才能與虛擬機器進行通訊。

若要為虛擬機器建立公用 IP 位址，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Creating public IP address...");
PublicIPAddress publicIPAddress = azure.publicIPAddresses()
    .define("myPublicIP")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withDynamicIP()
    .create();
```

### <a name="create-the-virtual-network"></a>建立虛擬網路

虛擬機器必須在[虛擬網路](../../virtual-network/virtual-networks-overview.md)的子網路中。

若要建立子網路和虛擬網路，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Creating virtual network...");
Network network = azure.networks()
    .define("myVN")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withAddressSpace("10.0.0.0/16")
    .withSubnet("mySubnet","10.0.0.0/24")
    .create();
```

### <a name="create-the-network-interface"></a>建立網路介面

虛擬機器需要網路介面以在虛擬網路上通訊。

若要建立網路介面，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Creating network interface...");
NetworkInterface networkInterface = azure.networkInterfaces()
    .define("myNIC")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetwork(network)
    .withSubnet("mySubnet")
    .withPrimaryPrivateIPAddressDynamic()
    .withExistingPrimaryPublicIPAddress(publicIPAddress)
    .create();
```

### <a name="create-the-virtual-machine"></a>建立虛擬機器

既然您已經建立所有支援的資源，您可以建立虛擬機器。

若要建立虛擬機器，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Creating virtual machine...");
VirtualMachine virtualMachine = azure.virtualMachines()
    .define("myVM")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetworkInterface(networkInterface)
    .withLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2012-R2-Datacenter")
    .withAdminUsername("azureuser")
    .withAdminPassword("Azure12345678")
    .withComputerName("myVM")
    .withExistingAvailabilitySet(availabilitySet)
    .withSize("Standard_DS1")
    .create();
Scanner input = new Scanner(System.in);
System.out.println("Press enter to get information about the VM...");
input.nextLine();
```

> [!NOTE]
> 本教學課程中會建立執行 Windows Server 作業系統版本的虛擬機器。 若要深入了解如何選取其他映像，請參閱 [使用 Windows PowerShell 和 Azure CLI 來瀏覽和選取 Azure 虛擬機器映像](../linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
> 
>

如果您想要使用現有的磁碟而不使用市集映像，請使用以下程式碼： 

```java
ManagedDisk managedDisk = azure.disks.define("myosdisk") 
    .withRegion(Region.US_EAST) 
    .withExistingResourceGroup("myResourceGroup") 
    .withWindowsFromVhd("https://mystorage.blob.core.windows.net/vhds/myosdisk.vhd") 
    .withSizeInGB(128) 
    .withSku(DiskSkuTypes.PremiumLRS) 
    .create(); 

azure.virtualMachines.define("myVM") 
    .withRegion(Region.US_EAST) 
    .withExistingResourceGroup("myResourceGroup") 
    .withExistingPrimaryNetworkInterface(networkInterface) 
    .withSpecializedOSDisk(managedDisk, OperatingSystemTypes.Windows) 
    .withExistingAvailabilitySet(availabilitySet) 
    .withSize(VirtualMachineSizeTypes.StandardDS1) 
    .create(); 
``` 

## <a name="perform-management-tasks"></a>執行管理工作

在虛擬機器的生命週期內，您可以執行一些管理工作，例如啟動、停止或刪除虛擬機器。 此外，您可以建立程式碼來自動執行重複或複雜的工作。

當您需要使用 VM 來執行任何操作時，必須取得其執行個體。 將此程式碼新增到 Main 方法的 try 區塊：

```java
VirtualMachine vm = azure.virtualMachines().getByResourceGroup("myResourceGroup", "myVM");
```

### <a name="get-information-about-the-vm"></a>取得 VM 的相關資訊

若要取得虛擬機器的相關資訊，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("hardwareProfile");
System.out.println("    vmSize: " + vm.size());
System.out.println("storageProfile");
System.out.println("  imageReference");
System.out.println("    publisher: " + vm.storageProfile().imageReference().publisher());
System.out.println("    offer: " + vm.storageProfile().imageReference().offer());
System.out.println("    sku: " + vm.storageProfile().imageReference().sku());
System.out.println("    version: " + vm.storageProfile().imageReference().version());
System.out.println("  osDisk");
System.out.println("    osType: " + vm.storageProfile().osDisk().osType());
System.out.println("    name: " + vm.storageProfile().osDisk().name());
System.out.println("    createOption: " + vm.storageProfile().osDisk().createOption());
System.out.println("    caching: " + vm.storageProfile().osDisk().caching());
System.out.println("osProfile");
System.out.println("    computerName: " + vm.osProfile().computerName());
System.out.println("    adminUserName: " + vm.osProfile().adminUsername());
System.out.println("    provisionVMAgent: " + vm.osProfile().windowsConfiguration().provisionVMAgent());
System.out.println("    enableAutomaticUpdates: " + vm.osProfile().windowsConfiguration().enableAutomaticUpdates());
System.out.println("networkProfile");
System.out.println("    networkInterface: " + vm.primaryNetworkInterfaceId());
System.out.println("vmAgent");
System.out.println("  vmAgentVersion: " + vm.instanceView().vmAgent().vmAgentVersion());
System.out.println("    statuses");
for(InstanceViewStatus status : vm.instanceView().vmAgent().statuses()) {
    System.out.println("    code: " + status.code());
    System.out.println("    displayStatus: " + status.displayStatus());
    System.out.println("    message: " + status.message());
    System.out.println("    time: " + status.time());
}
System.out.println("disks");
for(DiskInstanceView disk : vm.instanceView().disks()) {
    System.out.println("  name: " + disk.name());
    System.out.println("  statuses");
    for(InstanceViewStatus status : disk.statuses()) {
        System.out.println("    code: " + status.code());
        System.out.println("    displayStatus: " + status.displayStatus());
        System.out.println("    time: " + status.time());
    }
}
System.out.println("VM general status");
System.out.println("  provisioningStatus: " + vm.provisioningState());
System.out.println("  id: " + vm.id());
System.out.println("  name: " + vm.name());
System.out.println("  type: " + vm.type());
System.out.println("VM instance status");
for(InstanceViewStatus status : vm.instanceView().statuses()) {
    System.out.println("  code: " + status.code());
    System.out.println("  displayStatus: " + status.displayStatus());
}
System.out.println("Press enter to continue...");
input.nextLine();   
```

### <a name="stop-the-vm"></a>停止 VM

您可以停止虛擬機器並保留其所有的設定，但仍繼續計費，或您可以停止虛擬機器並將其解除配置。 當解除配置虛擬機器時，與其相關聯的所有資源也都會解除配置且其計費會結束。

若要停止虛擬機器而不解除配置，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Stopping vm...");
vm.powerOff();
System.out.println("Press enter to continue...");
input.nextLine();
```

如果您想要解除配置虛擬機器，請將 PowerOff 呼叫變更為下列程式碼︰

```java
vm.deallocate();
```

### <a name="start-the-vm"></a>啟動 VM

若要啟動虛擬機器，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Starting vm...");
vm.start();
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="resize-the-vm"></a>調整 VM 的大小

決定虛擬機器的大小時，應該考慮部署的許多層面。 如需詳細資訊，請參閱 [VM 大小](sizes.md)。  

若要變更虛擬機器的大小，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Resizing vm...");
vm.update()
    .withSize(VirtualMachineSizeTypes.STANDARD_DS2)
    .apply();
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="add-a-data-disk-to-the-vm"></a>將資料磁碟新增至 VM

若要將資料磁碟新增到大小為 2 GB、LUN 為 0 且快取類型為 ReadWrite 的虛擬機器，請將以下程式碼新增到 Main 方法中的 try 區塊：

```java
System.out.println("Adding data disk...");
vm.update()
    .withNewDataDisk(2, 0, CachingTypes.READ_WRITE)
    .apply();
System.out.println("Press enter to delete resources...");
input.nextLine();
```

## <a name="delete-resources"></a>刪除資源

由於您需要為在 Azure 中使用的資源付費，因此刪除不再需要的資源一律是理想的做法。 如果您想要刪除虛擬機器及所有支援的資源，您只需要刪除資源群組。

1. 若要刪除資源群組，請將以下程式碼新增到 Main 方法中的 try 區塊：
   
```java
System.out.println("Deleting resources...");
azure.resourceGroups().deleteByName("myResourceGroup");
```

2. 儲存 App.java 檔案。

## <a name="run-the-application"></a>執行應用程式

此主控台應用程式從開始到完成的完整執行應該需要五分鐘左右。

1. 若要執行應用程式，請使用此 Maven 命令：

    ```
    mvn compile exec:java
    ```

2. 在您按 **Enter** 以開始刪除資源之前，可以先花幾分鐘的時間來確認 Azure 入口網站中的資源建立情況。 請按一下部署狀態來查看該項部署的相關資訊。


## <a name="next-steps"></a>後續步驟
* 深入了解關於使用[適用於 JAVA 的 Azure 程式庫](https://docs.microsoft.com/java/azure/java-sdk-azure-overview)。

