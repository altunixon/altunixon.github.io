### Upgrade
1. Confirm current AADC settings
  - Get-ADSyncScheduler
  - Get-ADSyncAutoUpgrade
  - Open Azure AD Connect
  - Select Configure
  - Show current configuration
  - Azure AD Login GlobalAdmin@xxx.onmicrosoft.com:GAPassword
2. ? Backup .NET Config `C:\Windows\Microsoft.NET\Framework64\v4.0.xxxxx\Config\machine.config`
3. Backup sync rules
  - Open: Start\[Winkey\] > Azure AD Connect\[Start Folder\] > Synchronization Rule Editor\[Application\]
  - Select Custom Sync Rules > Export
  - Backup to custom-rules.ps1
4. Confirm current app version
  - Open ControlPanel `appwiz.cpl`
  - Microsoft Azure AD Connect > Version
5. Confirm database status MS SQL Server tools \(Dependencies\)
  - Open: Start\[Winkey\] > Microsoft SQL Server Tools 17\[Start Folder\] > Microsoft SQL Server Management Studio 17\[Application\]
  - Left panel > Databases > Right click ADSync > Properties
  - Take note of the `Storage Location` and `File name` AADC_SQL.ps1?
6. Disable Sync jobs in Task Scheduler \(Only on Active server, on Standby server these tasks are all disabled\)
  - Open: Task Scheduler\[Snapin\] > Left panel > Task Scheduler Library
  - Confirm: ACT_Azure_AD_Sync_Disable Task is **UNCHECKED**
  - Right click ACT_Azure_AD_Sync_Disable Task > Run now
  - Wait until it finishes
7. Perform upgrade
  - Note: Run installer with Enterprise Admin permission
  - Run AzureADConnect.msi > Select Upgrade
  - Provide valid User/Pass \(Global Admin\)
  - **Uncheck** Start the synchronization process when configuration completes
  - Click Upgrade
  - Wait til completes, takes ~3-5 minutes
  - Configuraion Complete > Exit
8. Reboot `shutdown -r -t 1`
9. Log back in
10. Repeat step 1, 4 Check Application Version after update
11. ? Restore .NET Config `fc machine.config.bak C:\Windows\Microsoft.NET\Framework64\v4.0.xxxxx\Config\machine.config`
12. Export current\(After upgrade\) rules: Repeat step 3
13. Check current database settings: Repeat step 5