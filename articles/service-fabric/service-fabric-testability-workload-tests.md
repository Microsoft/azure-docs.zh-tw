---
title: 模擬 Azure Service Fabric 應用程式中的錯誤 | Microsoft Docs
description: 如何針對非失誤性和失誤性失敗強化服務。
services: service-fabric
documentationcenter: .net
author: anmolah
manager: timlt
editor: ''
ms.assetid: 44af01f0-ed73-4c31-8ac0-d9d65b4ad2d6
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/15/2017
ms.author: anmola
ms.openlocfilehash: 3c075ac9642c7d050fc45ce6164071c9c733326e
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44051909"
---
# <a name="simulate-failures-during-service-workloads"></a>模擬服務工作負載期間的失敗案例
Azure Service Fabric 中的可測試性案例讓開發人員在處理個別錯誤時無需擔心。 不過還是有些案例，可能會需要明確的用戶端工作負載和失敗交錯。 用戶端工作負載和錯誤的交錯，可確保服務在發生失敗時確實執行某些動作。 假設可測試性提供的控制層級，可能是工作負載執行的精確時間點。 此應用程式中不同狀態的錯誤引發可以找到錯誤，並提升品質。

## <a name="sample-custom-scenario"></a>範例自訂案例
此測試說明了使用 [非失誤性和失誤性失敗](service-fabric-testability-actions.md#graceful-vs-ungraceful-fault-actions)商務工作負載交錯的案例。 為了獲得最佳結果，應該在服務作業或計算過程中引發錯誤。

我們來逐一解說某個服務範例，此範例公開四個工作負載：A、B、C 和 D。每個均對應至一組工作流程，且可能是計算、儲存體或混合。 為了簡單起見，我們會在範例中擷取出工作負載。 在此範例中執行的不同錯誤如下︰

* RestartNode︰模擬機器重新啟動的失誤性錯誤。
* RestartDeployedCodePackage︰模擬服務主機處理序當機。
* RemoveReplica︰模擬複本移除的非失誤性錯誤。
* MovePrimary︰模擬由 Service Fabric 負載平衡器所觸發的複本移動非失誤性錯誤。

```csharp
// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll.

using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        // Replace these strings with the actual version for your cluster and application.
        string clusterConnection = "localhost:19000";
        Uri applicationName = new Uri("fabric:/samples/PersistentToDoListApp");
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");

        Console.WriteLine("Starting Workload Test...");
        try
        {
            RunTestAsync(clusterConnection, applicationName, serviceName).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Workload Test failed: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Workload Test completed successfully.");
        return 0;
    }

    public enum ServiceWorkloads
    {
        A,
        B,
        C,
        D
    }

    public enum ServiceFabricFaults
    {
        RestartNode,
        RestartCodePackage,
        RemoveReplica,
        MovePrimary,
    }

    public static async Task RunTestAsync(string clusterConnection, Uri applicationName, Uri serviceName)
    {
        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);
        // Maximum time to wait for a service to stabilize.
        TimeSpan maxServiceStabilizationTime = TimeSpan.FromSeconds(120);

        // How many loops of faults you want to execute.
        uint testLoopCount = 20;
        Random random = new Random();

        for (var i = 0; i < testLoopCount; ++i)
        {
            var workload = SelectRandomValue<ServiceWorkloads>(random);
            // Start the workload.
            var workloadTask = RunWorkloadAsync(workload);

            // While the task is running, induce faults into the service. They can be ungraceful faults like
            // RestartNode and RestartDeployedCodePackage or graceful faults like RemoveReplica or MovePrimary.
            var fault = SelectRandomValue<ServiceFabricFaults>(random);

            // Create a replica selector, which will select a primary replica from the given service to test.
            var replicaSelector = ReplicaSelector.PrimaryOf(PartitionSelector.RandomOf(serviceName));
            // Run the selected random fault.
            await RunFaultAsync(applicationName, fault, replicaSelector, fabricClient);
            // Validate the health and stability of the service.
            await fabricClient.ServiceManager.ValidateServiceAsync(serviceName, maxServiceStabilizationTime);

            // Wait for the workload to finish successfully.
            await workloadTask;
        }
    }

    private static async Task RunFaultAsync(Uri applicationName, ServiceFabricFaults fault, ReplicaSelector selector, FabricClient client)
    {
        switch (fault)
        {
            case ServiceFabricFaults.RestartNode:
                await client.ClusterManager.RestartNodeAsync(selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RestartCodePackage:
                await client.ApplicationManager.RestartDeployedCodePackageAsync(applicationName, selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RemoveReplica:
                await client.ServiceManager.RemoveReplicaAsync(selector, CompletionMode.Verify, false);
                break;
            case ServiceFabricFaults.MovePrimary:
                await client.ServiceManager.MovePrimaryAsync(selector.PartitionSelector);
                break;
        }
    }

    private static Task RunWorkloadAsync(ServiceWorkloads workload)
    {
        throw new NotImplementedException();
        // This is where you trigger and complete your service workload.
        // Note that the faults induced while your service workload is running will
        // fault the primary service. Hence, you will need to reconnect to complete or check
        // the status of the workload.
    }

    private static T SelectRandomValue<T>(Random random)
    {
        Array values = Enum.GetValues(typeof(T));
        T workload = (T)values.GetValue(random.Next(values.Length));
        return workload;
    }
}
```
