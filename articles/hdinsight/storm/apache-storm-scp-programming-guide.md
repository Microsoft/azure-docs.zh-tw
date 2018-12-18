---
title: Azure HDInsight 中的 Storm 適用的 SCP.NET 程式設計指南
description: 了解如何使用 SCP.NET 建立以 .NET 為基礎的 Storm 拓撲，用於在 Azure HDInsight 中執行的 Storm。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonwhowell
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2016
ms.openlocfilehash: daf9efb2f9044ed1395b0f2aa3663c2471406e2f
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045459"
---
# <a name="scp-programming-guide"></a>SCP 程式設計指南
SCP 是一個用來建置即時、可靠、一致和高效能資料處理應用程式的平台。 建置在由 OSS 社群所設計的串流處理系統 [Apache Storm](http://storm.incubator.apache.org/) 之上。 Storm 由 Nathan Marz 所設計，由 Twitter 公開原始碼。 它採用 [Apache ZooKeeper](http://zookeeper.apache.org/)，這是另一個可發揮極可靠的分散式協調和狀態管理的 Apache 專案。 

SCP 專案不僅將 Storm 移植到 Windows 上，此專案也為 Windows 生態系統加入擴充和自訂功能。 擴充功能包括 .NET 開發人員體驗和程式庫，自訂功能包括以 Windows 為基礎的部署。 

擴充和自訂功能的設計讓我們不需要將 OSS 專案分岔，我們可以運用根據 Storm 而導出的生態系統。

## <a name="processing-model"></a>處理模型
SCP 中的資料模擬成連續的 Tuple 串流。 通常，Tuple 會先流進一些佇列，經過挑選，再由 Storm 拓撲內裝載的商業邏輯來轉換，最後，輸出以 Tuple 的形式傳遞至另一個 SCP 系統，或認可到存放區 (例如分散式檔案系統) 或資料庫 (例如 SQL Server)。

![佇列饋送資料 (饋送資料存放區) 以供處理的圖](./media/apache-storm-scp-programming-guide/queue-feeding-data-to-processing-to-data-store.png)

在 Storm 中，應用程式拓撲定義一份運算圖。 拓撲中的每個節點包含處理邏輯，節點之間的連結代表資料流程。 將輸入資料注入拓撲中的節點稱為 _Spout_，可用來編排資料。 輸入資料可能位於檔案記錄、交易式資料庫、系統效能計數器等。同時有輸入和輸出資料流程的節點稱為 _Bolt_，負責進行實際的資料篩選及挑選和彙總。

SCP 支援竭盡所能、至少一次和剛好一次這三種資料處理方法。 在分散式串流處理應用程式中，資料處理期間可能發生各種錯誤，例如網路中斷、機器故障或使用者程式碼錯誤等。至少一次的處理方法會在錯誤發生時自動重播相同資料，以確保所有資料至少處理一次。 至少一次的處理方法簡單又可靠，在許多應用程式中都很適合。 不過，當應用程式需要準確計數時 (只是舉例)，至少一次的處理方法就無法勝任，因為同樣的資料可能在應用程式拓撲中播放。 在此情況下，剛好一次的處理方法可確保即使資料可能重播和處理多次，結果也一定正確。

SCP 可讓 .NET 開發人員以 Java 虛擬機器 (JVM) 涵蓋 Storm，開發即時資料處理應用程式。 .NET 與 JVM 是透過 TCP 本機通訊端來進行通訊。 基本上，每個 Spout/Bolt 就是一對 .Net/Java 程序，而使用者邏輯在 .Net 程序中以外掛程式的方式運作。

若要根據 SCP 來建置資料處理應用程式，需要幾個步驟：

* 設計和實作 Spout 從佇列中拉進資料。
* 設計和實作 Bolt 來處理輸入資料，並將資料儲存至外部存放區，例如資料庫。
* 設計拓撲，然後提交並執行拓撲。 拓撲定義頂點及頂點之間的資料流程。 SCP 會採納拓撲規格，並部署到 Storm 叢集，而每個頂點就在一個邏輯節點上運作。 容錯移轉和調整由 Storm 工作排程器負責處理。

本文利用一些簡單的範例來逐步解說如何使用 SCP 建置資料處理應用程式。

## <a name="scp-plugin-interface"></a>SCP 外掛程式介面
SCP 外掛程式 (或應用程式) 是獨立式 EXE，可在開發階段於 Visual Studio 內執行，也可以在部署到實際執行環境之後插入 Storm 管線中。 撰寫 SCP 外掛程式就像是撰寫其他任何標準的 Windows 主控台應用程式一樣。 SCP.NET 平台為 spout/bolt 宣告一些介面，使用者外掛程式的程式碼應該實作這些介面。 此設計的主要目的是讓用者專注於自己的商業邏輯，將其他一切交給 SCP.NET 平台來處理。

使用者外掛程式的程式碼應該實作下列其中一個介面，視拓撲為交易式或非交易式而定，以及元作是 spout 或 bolt 而定。

* ISCPSpout
* ISCPBolt
* ISCPTxSpout
* ISCPBatchBolt

### <a name="iscpplugin"></a>ISCPPlugin
ISCPPlugin 是各種外掛程式的共同介面。 目前為虛擬介面。

    public interface ISCPPlugin 
    {
    }

### <a name="iscpspout"></a>ISCPSpout
ISCPSpout 為非交易式 spout 的介面。

     public interface ISCPSpout : ISCPPlugin                    
     {
         void NextTuple(Dictionary<string, Object> parms);         
         void Ack(long seqId, Dictionary<string, Object> parms);   
         void Fail(long seqId, Dictionary<string, Object> parms);  
     }

呼叫 `NextTuple()` 時，C\# 使用者程式碼可能發出一或多個 Tuple。 如果沒有資料可發出，此方法應該返回而不發出任何資料。 請注意，`NextTuple()`、`Ack()` 和 `Fail()` 都是在 C\# 程序的單一執行緒中放在密封迴圈內呼叫。 沒有 Tuple 可發出時，建議讓 NextTuple 短暫休息 (例如 10 毫秒)，不致於浪費太多 CPU。

只有當規格檔中啟用認可機制時，才會呼叫 `Ack()` 和 `Fail()`。 `seqId` 用來識別已認可或失敗的 Tuple。 因此，如果非交易式拓撲中啟用認可，則 Spout 中應該使用下列 emit 函數：

    public abstract void Emit(string streamId, List<object> values, long seqId); 

如果非交易式拓撲中不支援認可，則 `Ack()` 和 `Fail()` 可保持為空白函數。

這些函數中的 `parms` 輸入參數只是空的 Dictionary，保留供未來使用。

### <a name="iscpbolt"></a>ISCPBolt
ISCPBolt 為非交易式 bolt 的介面。

    public interface ISCPBolt : ISCPPlugin 
    {
    void Execute(SCPTuple tuple);           
    }

有新的 Tuple 可用時，會呼叫 `Execute()` 函數來處理它。

### <a name="iscptxspout"></a>ISCPTxSpout
ISCPTxSpout 為交易式 spout 的介面。

    public interface ISCPTxSpout : ISCPPlugin
    {
        void NextTx(out long seqId, Dictionary<string, Object> parms);  
        void Ack(long seqId, Dictionary<string, Object> parms);         
        void Fail(long seqId, Dictionary<string, Object> parms);        
    }

就像對應的非交易式節點一樣，`NextTx()`、`Ack()` 和 `Fail()` 也都是在 C\# 程序的單一執行緒中放在密封迴圈內呼叫。 沒有資料可發出時，建議讓 `NextTx` 短暫休息 (10 毫秒)，不致於浪費太多 CPU。

`NextTx()` 可呼叫來啟動新的交易，out 參數 `seqId` 用來識別交易，`Ack()` 和 `Fail()` 中也使用此參數。 在 `NextTx()`中，使用者可以發出資料給 Java 端。 資料會儲存在 ZooKeeper 中以支援重播。 因為 ZooKeeper 的容量有限，使用者在交易式 spout 中應該只發出中繼資料，而非大量資料。

Storm 會自動重播交易 (若失敗)，所以正常情況下應該不會呼叫 `Fail()` 。 但是，如果 SCP 可以檢查交易式 spout 所發出的元資料，則元資料無效時可以呼叫 `Fail()` 。

這些函數中的 `parms` 輸入參數只是空的 Dictionary，保留供未來使用。

### <a name="iscpbatchbolt"></a>ISCPBatchBolt
ISCPBatchBolt 為交易式 bolt 的介面。

    public interface ISCPBatchBolt : ISCPPlugin           
    {
        void Execute(SCPTuple tuple);
        void FinishBatch(Dictionary<string, Object> parms);  
    }

`Execute()` 。 `FinishBatch()` 。 `parms` 輸入參數保留供未來使用。

在交易式拓撲中，有一個重要的概念 – `StormTxAttempt`。 它有 `TxId` 和 `AttemptId` 兩個欄位。 `TxId` 用來識別特定的交易，在給定的交易中，如果交易失敗且重播，可能會嘗試很多次。 SCP.NET 會建立一個新的 ISCPBatchBolt 物件來處理每個 `StormTxAttempt`，就像 Storm 在 JAVA 的做法一樣。 此設計是為了支援平行交易處理。 使用者應該留意，如果交易嘗試完成，則會終結對應的 ISCPBatchBolt 物件，並回收其記憶體。

## <a name="object-model"></a>物件模型
SCP.NET 也提供一組簡單的關鍵物件供開發人員在程式設計中使用。 包括 **Context**、**StateStore** 和 **SCPRuntime**。 本節其餘部分將討論這些物件。

### <a name="context"></a>Context
Context 提供應用程式的執行環境。 每個 ISCPPlugin 執行個體 (ISCPSpout/ISCPBolt/ISCPTxSpout/ISCPBatchBolt) 都有一個對應的 Context 執行個體。 Context 提供的功能分成兩部分：(1) 靜態部分，供整個 C\# 程序使用，(2) 動態部分，僅供特定的 Context 執行個體使用。

### <a name="static-part"></a>靜態部分
    public static ILogger Logger = null;
    public static SCPPluginType pluginType;                      
    public static Config Config { get; set; }                    
    public static TopologyContext TopologyContext { get; set; }  

`Logger` 做為記錄用途。

`pluginType` 用來指出 C\# 程序的外掛程式類型。 如果 C\# 程序在本機測試模式 (無 Java) 中執行，則外掛程式類型為 `SCP_NET_LOCAL`。

    public enum SCPPluginType 
    {
        SCP_NET_LOCAL = 0,       
        SCP_NET_SPOUT = 1,       
        SCP_NET_BOLT = 2,        
        SCP_NET_TX_SPOUT = 3,   
        SCP_NET_BATCH_BOLT = 4  
    }

`Config` 可從 Java 端取得組態參數。 C\# 外掛程式初始化時，Java 端會傳回參數。 `Config` 參數分成兩部分：`stormConf` 和 `pluginConf`。

    public Dictionary<string, Object> stormConf { get; set; }  
    public Dictionary<string, Object> pluginConf { get; set; }  

`stormConf` 是由 Storm 定義的參數，`pluginConf` 是由 SCP 定義的參數。 例如︰

    public class Constants
    {
        … …

        // constant string for pluginConf
        public static readonly String NONTRANSACTIONAL_ENABLE_ACK = "nontransactional.ack.enabled";  

        // constant string for stormConf
        public static readonly String STORM_ZOOKEEPER_SERVERS = "storm.zookeeper.servers";           
        public static readonly String STORM_ZOOKEEPER_PORT = "storm.zookeeper.port";                 
    }

`TopologyContext` 可取得拓撲內容，這對多重平行處理的元件最實用。 下列是一個範例：

    //demo how to get TopologyContext info
    if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)                      
    {
        Context.Logger.Info("TopologyContext info:");
        TopologyContext topologyContext = Context.TopologyContext;                    
        Context.Logger.Info("taskId: {0}", topologyContext.GetThisTaskId());          
        taskIndex = topologyContext.GetThisTaskIndex();
        Context.Logger.Info("taskIndex: {0}", taskIndex);
        string componentId = topologyContext.GetThisComponentId();                    
        Context.Logger.Info("componentId: {0}", componentId);
        List<int> componentTasks = topologyContext.GetComponentTasks(componentId);  
        Context.Logger.Info("taskNum: {0}", componentTasks.Count);                    
    }

### <a name="dynamic-part"></a>動態部分
下列介面與特定的 Context 執行個體有關。 Context 執行個體由 SCP.NET 平台建立，並傳給使用者程式碼：

    // Declare the Output and Input Stream Schemas

    public void DeclareComponentSchema(ComponentStreamSchema schema);   

    // Emit tuple to default stream.
    public abstract void Emit(List<object> values);                   

    // Emit tuple to the specific stream.
    public abstract void Emit(string streamId, List<object> values);  

對於支援認可的非交易式 spout，已提供下列方法：

    // for non-transactional Spout which supports ack
    public abstract void Emit(string streamId, List<object> values, long seqId);  

對於支援認可的非交易式 bolt，應該明確呼叫 `Ack()` 或 `Fail()` 來處理收到的 Tuple。 發出新的 Tuple 時，也必須指定新 Tuple 的錨點。 已提供下列方法。

    public abstract void Emit(string streamId, IEnumerable<SCPTuple> anchors, List<object> values); 
    public abstract void Ack(SCPTuple tuple);
    public abstract void Fail(SCPTuple tuple);

### <a name="statestore"></a>StateStore
`StateStore` 提供元資料服務、單調數列產生和免等待協調。 高階分散式並行抽象可根據 `StateStore`來建置，包括分散式鎖定、分散式佇列、屏障和交易服務。

SCP 應用程式可使用 `State` 物件將某些資訊保存在 ZooKeeper 中，特別是針對交易式拓撲。 如此一來，如果交易式 spout 當機並重新啟動，就可從 ZooKeeper 擷取必要的資訊並重新啟動管線。

`StateStore` 物件主要有這些方法：

    /// <summary>
    /// Static method to retrieve a state store of the given path and connStr 
    /// </summary>
    /// <param name="storePath">StateStore Path</param>
    /// <param name="connStr">StateStore Address</param>
    /// <returns>Instance of StateStore</returns>
    public static StateStore Get(string storePath, string connStr);

    /// <summary>
    /// Create a new state object in this state store instance
    /// </summary>
    /// <returns>State from StateStore</returns>
    public State Create();

    /// <summary>
    /// Retrieve all states that were previously uncommitted, excluding all aborted states 
    /// </summary>
    /// <returns>Uncommited States</returns>
    public IEnumerable<State> GetUnCommitted();

    /// <summary>
    /// Get all the States in the StateStore
    /// </summary>
    /// <returns>All the States</returns>
    public IEnumerable<State> States();

    /// <summary>
    /// Get state or registry object
    /// </summary>
    /// <param name="info">Registry Name(Registry only)</param>
    /// <typeparam name="T">Type, Registry or State</typeparam>
    /// <returns>Return Registry or State</returns>
    public T Get<T>(string info = null);

    /// <summary>
    /// List all the committed states
    /// </summary>
    /// <returns>Registries contain the Committed State </returns> 
    public IEnumerable<Registry> Commited();

    /// <summary>
    /// List all the Aborted State in the StateStore
    /// </summary>
    /// <returns>Registries contain the Aborted State</returns>
    public IEnumerable<Registry> Aborted();

    /// <summary>
    /// Retrieve an existing state object from this state store instance 
    /// </summary>
    /// <returns>State from StateStore</returns>
    /// <typeparam name="T">stateId, id of the State</typeparam>
    public State GetState(long stateId)

`State` 物件主要有這些方法：

    /// <summary>
    /// Set the status of the state object to commit 
    /// </summary>
    public void Commit(bool simpleMode = true); 

    /// <summary>
    /// Set the status of the state object to abort 
    /// </summary>
    public void Abort();

    /// <summary>
    /// Put an attribute value under the give key 
    /// </summary>
    /// <param name="key">Key</param> 
    /// <param name="attribute">State Attribute</param> 
    public void PutAttribute<T>(string key, T attribute); 

    /// <summary>
    /// Get the attribute value associated with the given key 
    /// </summary>
    /// <param name="key">Key</param> 
    /// <returns>State Attribute</returns>               
    public T GetAttribute<T>(string key);                    

在 `Commit()` 方法中，當 simpleMode 設為 true 時，就會在 ZooKeeper 中刪除對應的 ZNode。 否則會刪除目前的 ZNode，並在 COMMITTED\_PATH 中加入新的節點。

### <a name="scpruntime"></a>SCPRuntime
SCPRuntime 提供下列兩個方法：

    public static void Initialize();

    public static void LaunchPlugin(newSCPPlugin createDelegate);  

`Initialize()` 用來初始化 SCP 執行階段環境。 在此方法中，C\# 程序會連接到 JAVA 端，並取得組態參數和拓撲內容。

`LaunchPlugin()` 用來啟動訊息處理迴圈。 在此迴圈中，C\# 外掛程式會從 JAVA 端接收訊息 (包括 Tuple 和控制訊號)，然後處理訊息，也許會呼叫使用者程式碼提供的介面方法。 `LaunchPlugin()` 方法的輸入參數是委派，可傳回一個實作 ISCPSpout/IScpBolt/ISCPTxSpout/ISCPBatchBolt 介面的物件。

    public delegate ISCPPlugin newSCPPlugin(Context ctx, Dictionary\<string, Object\> parms); 

在 ISCPBatchBolt 中，我們可以從 `parms` 取得 `StormTxAttempt`，用以判斷是否為重播的嘗試。 重播嘗試的檢查通常是在認可 bolt 上完成，`HelloWorldTx` 範例中會示範。

一般而言，SCP 外掛程式可能在以下兩種模式中執行：

1. 本機測試模式：在此模式中，SCP 外掛程式 (C\# 使用者程式碼) 在開發階段是在 Visual Studio 內執行。 `LocalContext` ，它提供方法將發出的 Tuple 序列化到本機檔案，再讀回到記憶體中。
   
        public interface ILocalContext
        {
            List\<SCPTuple\> RecvFromMsgQueue();
            void WriteMsgQueueToFile(string filepath, bool append = false);  
            void ReadFromFileToMsgQueue(string filepath);                    
        }
2. 標準模式：在此模式中，SCP 外掛程式由 storm java 程序啟動。
   
    以下是啟動 SCP 外掛程式的範例：
   
        namespace Scp.App.HelloWorld
        {
        public class Generator : ISCPSpout
        {
            … …
            public static Generator Get(Context ctx, Dictionary<string, Object> parms)
            {
            return new Generator(ctx);
            }
        }
   
        class HelloWorld
        {
            static void Main(string[] args)
            {
            /* Setting the environment variable here can change the log file name */
            System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "HelloWorld");
   
            SCPRuntime.Initialize();
            SCPRuntime.LaunchPlugin(new newSCPPlugin(Generator.Get));
            }
        }
        }

## <a name="topology-specification-language"></a>拓撲規格語言
SCP 拓撲規格是特定領域的語言，用來描述和設定 SCP 拓撲。 它是以 Storm 為基礎的 Clojure DSL (<http://storm.incubator.apache.org/documentation/Clojure-DSL.html>) 並由 SCP 進行擴充。

拓撲規格可透過 ***runspec*** 命令直接提交給 storm 叢集來執行。

SCP.NET 已增加下列函式來定義交易拓撲：

| **新函數** | **參數** | **說明** |
| --- | --- | --- |
| **tx-topolopy** |topology-name<br />spout-map<br />bolt-map |以拓撲名稱、&nbsp;spout 定義對應和 bolt 定義對應來定義交易式拓撲 |
| **scp-tx-spout** |exec-name<br />args<br />fields |定義交易式 spout。 它會使用 ***args*** 搭配 ***exec-name*** 來執行應用程式。<br /><br />***fields*** 是 spout 的輸出欄位 |
| **scp-tx-batch-bolt** |exec-name<br />args<br />fields |定義交易式批次 Bolt。 它會使用 ***args*** 搭配 ***exec-name*** 來執行應用程式。<br /><br />fields 是 bolt 的輸出欄位。 |
| **scp-tx-commit-bolt** |exec-name<br />args<br />fields |定義交易式認可 Bolt。 它會使用 ***args*** 搭配 ***exec-name*** 來執行應用程式。<br /><br />***fields*** 是 bolt 的輸出欄位 |
| **nontx-topolopy** |topology-name<br />spout-map<br />bolt-map |以拓撲名稱、&nbsp;spout 定義對應和 bolt 定義對應來定義非交易式拓撲 |
| **scp-spout** |exec-name<br />args<br />fields<br />parameters |定義非交易式 spout。 它會使用 ***args*** 搭配 ***exec-name*** 來執行應用程式。<br /><br />***fields*** 是 spout 的輸出欄位<br /><br />***parameters*** 為選用，使用它來指定一些參數，例如 "nontransactional.ack.enabled"。 |
| **scp-bolt** |exec-name<br />args<br />fields<br />parameters |定義非交易式 Bolt。 它會使用 ***args*** 搭配 ***exec-name*** 來執行應用程式。<br /><br />***fields*** 是 bolt 的輸出欄位<br /><br />***parameters*** 為選用，使用它來指定一些參數，例如 "nontransactional.ack.enabled"。 |

SCP.NET 已定義下列關鍵字：

| **關鍵字** | **說明** |
| --- | --- |
| **:name** |定義拓撲名稱 |
| **:topology** |使用先前的函數和內建函數來定義拓撲。 |
| **:p** |為每個 spout 或 bolt 定義平行處理提示。 |
| **:config** |定義設定參數或更新現有的設定參數 |
| **:schema** |定義串流的結構描述。 |

常用的參數則有：

| **參數** | **說明** |
| --- | --- |
| **"plugin.name"** |C# 外掛程式的 exe 檔名 |
| **"plugin.args"** |外掛程式引數 |
| **"output.schema"** |輸出結構描述 |
| **"nontransactional.ack.enabled"** |非交易式拓撲是否啟用認可 |

runspec 命令會隨著程式碼一起部署，用法如下：

    .\bin\runSpec.cmd
    usage: runSpec [spec-file target-dir [resource-dir] [-cp classpath]]
    ex: runSpec examples\HelloWorld\HelloWorld.spec specs examples\HelloWorld\Target

***resource-dir*** 參數是選擇性，想要插入 C\# 應用程式時需要指定它，此目錄將包含應用程式、相依性和組態。

***classpath*** 參數也是選擇性。 用來指定 Java 類別路徑 (如果規格檔包含 Java Spout 或 Bolt)。

## <a name="miscellaneous-features"></a>其他功能
### <a name="input-and-output-schema-declaration"></a>輸入和輸出結構描述宣告
使用者可以在 C\# 程序中發出 Tuple，平台必須將 Tuple 序列化為 byte，並傳送至 Java 端，Storm 會將此 Tuple 傳送至目標。 同時，在下游元件中，C\# 程序會接收 java 端傳回的 Tuple，由平台將它轉換成原始類型，而所有這些操作都由平台在幕後進行。

為了支援序列化和還原序列化，使用者程式碼需要宣告輸入和輸出的結構描述。

輸入/輸出資料流結構描述定義為字典。 金鑰為 StreamId。 此值是資料行的類型。 元件可以宣告多重串流。

    public class ComponentStreamSchema
    {
        public Dictionary<string, List<Type>> InputStreamSchema { get; set; }
        public Dictionary<string, List<Type>> OutputStreamSchema { get; set; }
        public ComponentStreamSchema(Dictionary<string, List<Type>> input, Dictionary<string, List<Type>> output)
        {
            InputStreamSchema = input;
            OutputStreamSchema = output;
        }
    }


在 Context 物件中，我們增加下列 API：

    public void DeclareComponentSchema(ComponentStreamSchema schema)

開發者必須確定發出的 Tuple 遵守該串流所定義的結構描述，否則系統會擲回執行階段例外狀況。

### <a name="multi-stream-support"></a>多重串流支援
SCP 支援使用者程式碼同時發出或接收多個不同串流。 此支援反映在 Context 物件中，因為 Emit 方法接受一個選擇性串流 ID 參數。

SCP.NET Context 物件中已增加兩個方法。 用以發出一或多個 Tuple 來指定 StreamId。 StreamId 是字串，在 C\# 與拓撲定義規格中必須一致。

        /* Emit tuple to the specific stream. */
        public abstract void Emit(string streamId, List<object> values);

        /* for non-transactional Spout only */
        public abstract void Emit(string streamId, List<object> values, long seqId);

發出給不存在的串流會造成執行階段例外狀況。

### <a name="fields-grouping"></a>欄位分組
Strom 中內建的欄位群組功能在 SCP.NET 中無法正常運作。 在 Java Proxy 端，所有欄位資料類型實際上為 byte[]，而欄位群組會使用 byte[] 物件雜湊碼來執行群組。 byte[] 物件雜湊碼是此物件在記憶體中的位址。 因此，共用相同內容但不是相同位址的兩個 byte[] 物件，分組會錯誤。

SCP.NET 增加一個自訂的分組方法，它會使用 byte[] 的內容來執行分組。 在 **SPEC** 檔案中，語法如下：

    (bolt-spec
        {
            "spout_test" (scp-field-group :non-tx [0,1])
        }
        …
    )


在這裡，

1. “scp-field-group” 表示「SCP 實作的自訂欄位分組」。
2. “:tx” 或 “:non-tx” 表示是否為交易式拓撲。 我們需要此資訊，因為 tx 和非 tx 拓撲中的起始索引不同。
3. [0,1] 表示欄位識別碼的雜湊集，從 0 開始。

### <a name="hybrid-topology"></a>混合式拓撲
原生 Storm 是以 Java 撰寫。 且 SCP.Net 已經增強，讓 C\# 開發者可以撰寫 C\# 程式碼來處理其商業邏輯。 但它也支援混合式拓撲，不僅包含 C\# spout/bolt，也包含 Java Spout/Bolt。

### <a name="specify-java-spoutbolt-in-spec-file"></a>在規格檔中指定 Java Spout/Bolt
在規格檔中，"scp-spout" 和 "scp-bolt" 也可用來指定 Java Spout 和 Bolt，如下列範例所示：

    (spout-spec 
      (microsoft.scp.example.HybridTopology.Generator.)           
      :p 1)

其中 `microsoft.scp.example.HybridTopology.Generator` 是 Java Spout 類別的名稱。

### <a name="specify-java-classpath-in-runspec-command"></a>在 runSpec 命令中指定 Java 類別路徑
如果您要提交包含 Java Spout 或 Bolt 的拓撲，則必須先編譯 Java Spout 或 Bolt 並取得 Jar 檔案。 然後，在提交拓撲時，應該指定包含這些 Jar 檔案的 java 類別路徑。 下列是一個範例：

    bin\runSpec.cmd examples\HybridTopology\HybridTopology.spec specs examples\HybridTopology\net\Target -cp examples\HybridTopology\java\target\*

在這裡，**examples\\HybridTopology\\java\\target\\** 是包含 Java Spout/Bolt Jar 檔案的資料夾。

### <a name="serialization-and-deserialization-between-java-and-c"></a>Java 與 C\# 之間的序列化和還原序列化
SCP 元件包含 Java 和 C\# 端。 為了與原生 Java Spout/Bolt 互動，必須在 Java 和 C\# 端之間進行序列化/還原序列化，如下圖所示。

![Java 元件傳送至 SCP 元件再傳送至 Java 元件的圖](./media/apache-storm-scp-programming-guide/java-compent-sending-to-scp-component-sending-to-java-component.png)

1. **Java 端序列化和 C\# 端還原序列化**
   
   首次，我們提供 Java 端序列化和 C\# 端還原序列化的預設實作。 Java 端的序列化方法可以在 SPEC 檔案中指定：
   
       (scp-bolt
           {
               "plugin.name" "HybridTopology.exe"
               "plugin.args" ["displayer"]
               "output.schema" {}
               "customized.java.serializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer"]
           })
   
   C\# 端的還原序列化方法應該在 C\# 使用者程式碼中指定：
   
       Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
       inputSchema.Add("default", new List<Type>() { typeof(Person) });
       this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));
       this.ctx.DeclareCustomizedDeserializer(new CustomizedInteropJSONDeserializer());            
   
   只要資料類型不要太複雜，此預設實作應該能夠因應大多數的情況。 在某些情況下，由於使用者資料類型太複雜，或因為預設實作的效能不符合使用者的需求，使用者可以插入自己的實作。
   
   Java 端的序列化介面定義為：
   
       public interface ICustomizedInteropJavaSerializer {
           public void prepare(String[] args);
           public List<ByteBuffer> serialize(List<Object> objectList);
       }
   
   C\# 端的還原序列化介面定義為：
   
   公用介面 ICustomizedInteropCSharpDeserializer
   
       public interface ICustomizedInteropCSharpDeserializer
       {
           List<Object> Deserialize(List<byte[]> dataList, List<Type> targetTypes);
       }
2. **C 端序列化\#和 JAVA 端還原序列化**
   
   應該在 C\# 使用者程式碼中指定 C\# 端的還原序列化方法：
   
       this.ctx.DeclareCustomizedSerializer(new CustomizedInteropJSONSerializer()); 
   
   應該在 SPEC 檔案中指定 Java 端的還原序列化方法：
   
     (scp-spout
   
       {
         "plugin.name" "HybridTopology.exe"
         "plugin.args" ["generator"]
         "output.schema" {"default" ["person"]}
         "customized.java.deserializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer" "microsoft.scp.example.HybridTopology.Person"]
       })
   
   其中 "microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer" 是 Deserializer (還原序列化程式) 的名稱，"microsoft.scp.example.HybridTopology.Person" 是資料還原序列化的目標類別。
   
   使用者也可以外掛其自己實作的 C\# 序列化程式和 JAVA Deserializer。 此程式碼是 C\# 序列化程式的介面︰
   
       public interface ICustomizedInteropCSharpSerializer
       {
           List<byte[]> Serialize(List<object> dataList);
       }
   
   此程式碼是 JAVA Deserializer 的介面︰
   
       public interface ICustomizedInteropJavaDeserializer {
           public void prepare(String[] targetClassNames);
           public List<Object> Deserialize(List<ByteBuffer> dataList);
       }

## <a name="scp-host-mode"></a>SCP 主機模式
在此模式中，使用者可以將程式碼編譯成 DLL，並使用 SCP 提供的 SCPHost.exe 來提交拓撲。 規格檔如此程式碼所示：

    (scp-spout
      {
        "plugin.name" "SCPHost.exe"
        "plugin.args" ["HelloWorld.dll" "Scp.App.HelloWorld.Generator" "Get"]
        "output.schema" {"default" ["sentence"]}
      })

其中，`plugin.name` 指定為 SCP SDK 所提供的 `SCPHost.exe`。 SCPHost.exe 接受三個參數：

1. 第一個是 DLL 名稱，在此範例中為 `"HelloWorld.dll"` 。
2. 第二個是類別名稱，在此範例中為 `"Scp.App.HelloWorld.Generator"` 。
3. 第三個是 public static 方法的名稱，可叫用來取得 ISCPPlugin 的執行個體。

在主機模式中，使用者程式碼會編譯成 DLL，供 SCP 平台叫用。 SCP 平台可以完全掌控整個處理邏輯。 因此，建議客戶在 SCP 主機模式中提交拓撲，因為這樣可以簡化開發過程，讓我們有更大的彈性，在後續版本中也能有更高的回溯相容性。

## <a name="scp-programming-examples"></a>SCP 程式設計範例
### <a name="helloworld"></a>HelloWorld
**HelloWorld** 是一個體驗 SCP.Net 的簡單範例。 它使用非交易拓撲，具有一個名為 **generator** 的 spout，以及名為 **splitter** 和 **counter** 的兩個 bolt。 spout **generator** 會隨機產生句子，並發出這些句子給 **splitter**。 bolt **splitter** 會將這些句子分割成單字，再發出這些單字給 **counter** bolt。 bolt "counter" 使用字典來記錄每個單字出現的次數。

此範例有兩個規格檔：**HelloWorld.spec** 和 **HelloWorld\_EnableAck.spec**。 在 C\# 程式碼中，可從 Java 端取得 pluginConf 來檢查認可是否已啟用。

    /* demo how to get pluginConf info */
    if (Context.Config.pluginConf.ContainsKey(Constants.NONTRANSACTIONAL_ENABLE_ACK))
    {
        enableAck = (bool)(Context.Config.pluginConf[Constants.NONTRANSACTIONAL_ENABLE_ACK]);
    }
    Context.Logger.Info("enableAck: {0}", enableAck);

在 spout 中，如果認可已啟用，則會使用字典來快取尚未認可的 Tuple。 如果呼叫 Fail()，則會重播失敗的 Tuple：

    public void Fail(long seqId, Dictionary<string, Object> parms)
    {
        Context.Logger.Info("Fail, seqId: {0}", seqId);
        if (cachedTuples.ContainsKey(seqId))
        {
            /* get the cached tuple */
            string sentence = cachedTuples[seqId];

            /* replay the failed tuple */
            Context.Logger.Info("Re-Emit: {0}, seqId: {1}", sentence, seqId);
            this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), seqId);
        }
        else
        {
            Context.Logger.Warn("Fail(), can't find cached tuple for seqId {0}!", seqId);
        }
    }

### <a name="helloworldtx"></a>HelloWorldTx
**HelloWorldTx** 範例示範如何實作交易式拓撲。 它有一個名為 **generator** 的 spout，一個名為 **partial-count** 的批次 bolt，以及一個名為 **count-sum** 的認可 bolt。 另外還有三個預先建立的 txt 檔案︰**DataSource0.txt**、**DataSource1.txt**、**DataSource2.txt**。

在每個交易中，**generator** spout 會從預先建立的三個檔案中隨機選取兩個檔案，然後發出這兩個檔名給 **partial-count** bolt。 **partial-count** bolt 會先從收到的 Tuple 中取得檔名，然後開啟檔案並計算此檔案中的字數，最後再發出字數給 **count-sum** bolt。 **count-sum** bolt 將計算總數。

為了符合**剛好一次**語意，認可 bolt **count-sum** 需要判斷它是否為重播的交易。 在此範例中，它有一個靜態成員變數：

    public static long lastCommittedTxId = -1; 

建立 ISCPBatchBolt 執行個體時，它會從輸入參數中取得 `txAttempt`：

    public static CountSum Get(Context ctx, Dictionary<string, Object> parms)
    {
        /* for transactional topology, we can get txAttempt from the input parms */
        if (parms.ContainsKey(Constants.STORM_TX_ATTEMPT))
        {
            StormTxAttempt txAttempt = (StormTxAttempt)parms[Constants.STORM_TX_ATTEMPT];
            return new CountSum(ctx, txAttempt);
        }
        else
        {
            throw new Exception("null txAttempt");
        }
    }

如果不是重播的交易，呼叫 `FinishBatch()` 時，將會更新 `lastCommittedTxId`。

    public void FinishBatch(Dictionary<string, Object> parms)
    {
        /* judge whether it is a replayed transaction? */
        bool replay = (this.txAttempt.TxId <= lastCommittedTxId);

        if (!replay)
        {
            /* If it is not replayed, update the totalCount and lastCommittedTxId value */
            totalCount = totalCount + this.count;
            lastCommittedTxId = this.txAttempt.TxId;
        }
        … …
    }


### <a name="hybridtopology"></a>HybridTopology
此拓撲包含 Java Spout 和 C\# Bolt。 它採用 SCP 平台提供的預設序列化和還原序列化實作。 請參閱 **examples\\HybridTopology** 資料夾中的 **HybridTopology.spec**，以取得規格檔詳細資料，並查看 **SubmitTopology.bat** 了解如何指定 JAVA 類別路徑。

### <a name="scphostdemo"></a>SCPHostDemo
此範例在本質上與 HelloWorld 相同。 唯一的差別在於使用者程式碼是編譯成 DLL，且使用 SCPHost.exe 提交拓撲。 如需詳細說明，請參閱＜SCP 主機模式＞一節。

## <a name="next-steps"></a>後續步驟
有關使用 SCP 建立之 Storm 拓撲的詳細資訊，請參閱下列各文︰

* [使用 Visual Studio 開發 Apache Storm on HDInsight 的 C# 拓撲](apache-storm-develop-csharp-visual-studio-topology.md)
* [利用 Storm on HDInsight 處理 Azure 事件中心的事件](apache-storm-develop-csharp-event-hub-topology.md)
* [使用 Storm on HDInsight 處理事件中心的車輛感應器資料](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/IotExample)
* [從 Azure 事件中樞擷取、轉換及載入 (ETL) 至 HBase](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample)
