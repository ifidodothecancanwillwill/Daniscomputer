# Function to handle errors gracefully
function Handle-Error {
    param ($ErrorMessage)
    Write-Host "An error occurred: $ErrorMessage" -ForegroundColor Red
}

# Disable Startup Programs
try {
    Get-CimInstance Win32_StartupCommand | Where-Object { $_.Location -eq "HKLM" -or $_.Location -eq "HKCU" } | Remove-CimInstance -ErrorAction Stop
    Write-Host "Startup programs disabled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to disable startup programs."
}

# Set High-Performance Power Plan
try {
    powercfg -setactive SCHEME_MAX
    Write-Host "High-Performance power plan set successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to set High-Performance power plan."
}

# Perform Disk Cleanup
try {
    cleanmgr /sagerun:1
    Write-Host "Disk cleanup executed successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to perform disk cleanup."
}

# Disable Unnecessary Windows Services (SysMain and Windows Search)
try {
    Get-Service | Where-Object { $_.DisplayName -like "Windows Search" -or $_.DisplayName -like "SysMain" } | Stop-Service -Force -ErrorAction SilentlyContinue
    Get-Service | Where-Object { $_.DisplayName -like "Windows Search" -or $_.DisplayName -like "SysMain" } | Set-Service -StartupType Disabled -ErrorAction SilentlyContinue
    Write-Host "Unnecessary services disabled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to disable unnecessary services."
}

# Adjust Virtual Memory
try {
    $cs = Get-WmiObject Win32_ComputerSystem
    $cs.AutomaticManagedPagefile = $False
    $cs.Put()
    Set-WmiInstance -Class Win32_PageFileSetting -Arguments @{Name = "C:\pagefile.sys"; InitialSize = 4096; MaximumSize = 8192} -ErrorAction SilentlyContinue
    Write-Host "Virtual memory adjusted successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to adjust virtual memory."
}

# Remove Unnecessary Registry Entries
try {
    Remove-ItemProperty -Path HKCU:\Software\Microsoft\Windows\CurrentVersion\Run -Name "UnwantedApp" -ErrorAction SilentlyContinue
    Write-Host "Unnecessary registry entries removed successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to remove unnecessary registry entries."
}

# Uninstall Bloatware
try {
    Get-AppxPackage | Where-Object { $_.Name -like "*bing*" -or $_.Name -like "*games*" } | Remove-AppxPackage -ErrorAction SilentlyContinue
    Write-Host "Bloatware uninstalled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to uninstall bloatware."
}

# Disable Animations for Speed
try {
    Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "VisualFXSetting" -Value 2 -ErrorAction SilentlyContinue
    Write-Host "Animations disabled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to disable animations."
}

Write-Host "All Optimization Tasks Completed. Thank you." -ForegroundColor Green
