1. 將安裝程式複製到您想要保護之伺服器上的本機資料夾 (例如 C:\Temp)。 請以網域系統管理員身分在命令提示字元執行下列命令：

  ```
  cd C:\Temp
  ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
  MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
  cd C:\Temp\Extracted.
  ```
2. 若要安裝行動服務，請執行下列命令：

  ```
  UnifiedAgent.exe /Role "MS" /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
  ```
3. 現在必須向組態伺服器註冊代理程式。

  ```
  cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
  UnifiedAgentConfigurator.exe  /CSEndPoint <CSIP> /PassphraseFilePath <PassphraseFilePath>
  ```

#### <a name="mobility-service-installer-command-line-arguments"></a>行動服務安裝程式命令列引數

```
Usage :
UnifiedAgent.exe /Role <MS|MT> /InstallLocation <Install Location> /Platform “VmWare” /Silent
```

| 參數|類型|說明|可能的值|
|-|-|-|-|
|/Role|強制|指定應該安裝行動服務 (MS) 還是應該安裝主要目標 (MT)。|MS </br> MT|
|/InstallLocation|選用|行動服務的安裝位置。|電腦上的任何資料夾|
|/Platform|強制|指定要安裝行動服務的平台。 </br> </br>- **VMware**︰如果您要在「VMware vSphere ESXi 主機」、「Hyper-V 主機」和「實體伺服器」上執行的 VM 上安裝行動服務，請使用此值。 </br> - **Azure**︰如果您要在 Azure IaaS VM 上安裝代理程式，請使用此值。 | VMware </br> Azure|
|/Silent|選用|指定要以無訊息模式執行安裝程式。| N/A|

>[!TIP]
> 可以在 %ProgramData%\ASRSetupLogs\ASRUnifiedAgentInstaller.log 下找到設定記錄。

#### <a name="mobility-service-registration-command-line-arguments"></a>行動服務註冊命令列引數

```
Usage :
UnifiedAgentConfigurator.exe  /CSEndPoint <CSIP> /PassphraseFilePath <PassphraseFilePath>
```

  | 參數|類型|說明|可能的值|
  |-|-|-|-|
  |/CSEndPoint |強制|組態伺服器的 IP 位址| 任何有效的 IP 位址|
  |/PassphraseFilePath|強制|複雜密碼的位置 |任何有效的 UNC 或本機檔案路徑|


>[!TIP]
> 可以在 %ProgramData%\ASRSetupLogs\ASRUnifiedAgentConfigurator.log 之下找到 Agent Configuration 記錄。
