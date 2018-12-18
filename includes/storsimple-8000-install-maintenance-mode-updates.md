### <a name="install-maintenance-mode-updates-via-windows-powershell-for-storsimple"></a>透過適用於 StorSimple 的 Windows PowerShell 安裝維護模式更新

當您將維護模式更新將套用至 StorSimple 裝置時，所有 I/O 要求都會暫停。 靜態隨機存取記憶體 (NVRAM) 之類的服務或叢集服務都會停止。 這兩個控制站會在您進入或結束此模式時重新啟動。 當您結束此模式時，所有的服務都會繼續執行，而且健康情況良好。 (這可能需要數分鐘的時間)。

> [!IMPORTANT]
> * 進入維護模式之前，請在 Azure 入口網站中確認這兩個裝置控制器的健康情況良好。 如果控制站的健康情況不好，[請連絡 Microsoft 支援服務](../articles/storsimple/storsimple-8000-contact-microsoft-support.md)以進行後續步驟。
> * 當您處於維護模式時，需要先更新某一個控制站，然後再更新另一個控制站。

1. 使用 PuTTY 連接到序列主控台。 請依照[使用 PuTTy 連接到序列主控台](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console)中的詳細指示執行作業。 在命令提示字元中，按 **Enter**鍵。 選取選項 1 [以完整存取權登入]。

2. 若要使控制器處於維護模式，請輸入：
    
    `Enter-HcsMaintenanceMode`

    這兩個控制器就會重新啟動以進入維護模式。

3. 安裝維護模式更新。 輸入：

    `Start-HcsUpdate`

    系統會提示您進行確認。 當您確認更新之後，它們會安裝於您目前正在存取的控制站上。 安裝更新之後，隨即重新啟動控制站。

4. 監視更新的狀態。 當目前的控制器正在更新且無法處理任何其他命令時，登入對等控制器。 輸入：

    `Get-HcsUpdateStatus`

    如果 `RunInProgress` 是 `True`，表示更新仍在進行中。 如果 `RunInProgress` 是 `False`，表示已完成更新。

5. 成功套用磁碟韌體更新並重新啟動更新的控制器之後，請確認磁碟韌體版本。 在更新的控制器上，輸入：

    `Get-HcsFirmwareVersion`
   
    預期的磁碟韌體版本為：`XMGJ, XGEG, KZ50, F6C2, VR08, N003, 0107`

6. 結束維護模式。 針對每個裝置控制器輸入以下命令：

    `Exit-HcsMaintenanceMode`

    當您離開維護模式時，控制器便會重新啟動。

7. 返回 Azure 入口網站。 入口網站可能有 24 小時的時間不會顯示您已安裝維護模式更新。