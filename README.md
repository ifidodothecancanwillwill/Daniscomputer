# Disable Startup Programs
Get-CimInstance Win32_StartupCommand | Where-Object { $_.Location -eq "HKLM" -or $_.Location -eq "HKCU" } | ForEach-Object { $_ | Remove-CimInstance }

# Set High-Performance Power Plan
powercfg -setactive SCHEME_MAX

# Perform Disk Cleanup
Start-Process cleanmgr -ArgumentList "/sagerun:1" -Wait

# Disable Unnecessary Windows Services (SysMain and Windows Search)
Get-Service | Where-Object { $_.DisplayName -like "Windows Search" -or $_.DisplayName -like "SysMain" } | Stop-Service -Force
Set-Service -Name "SysMain" -StartupType Disabled
Set-Service -Name "wsearch" -StartupType Disabled

# Adjust Virtual Memory
Set-WmiInstance -Path Win32_ComputerSystem -Arguments @{AutomaticManagedPagefile = $False}
Set-WmiInstance -Class Win32_PageFileSetting -Arguments @{Name = "C:\pagefile.sys"; InitialSize = 4096; MaximumSize = 8192}

# Remove Unnecessary Registry Entries
Remove-ItemProperty -Path HKCU:\Software\Microsoft\Windows\CurrentVersion\Run -Name "UnwantedApp" -ErrorAction SilentlyContinue

# Uninstall Bloatware
Get-AppxPackage | Where-Object { $_.Name -like "*bing*" -or $_.Name -like "*games*" } | Remove-AppxPackage

# Disable Animations for Speed
Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "VisualFXSetting" -Value 2

# Optional: Enable Restart Required for System Settings
Write-Host "Some changes may require a system restart to take full effect." -ForegroundColor Yellow
