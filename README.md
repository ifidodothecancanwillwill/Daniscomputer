# Display a welcome message
Add-Type -AssemblyName Microsoft.VisualBasic
[Microsoft.VisualBasic.Interaction]::MsgBox("Hi Dani, click OK to start!", "OKOnly", "Optimization Script")

# Function to handle errors gracefully
function Handle-Error {
    param ($ErrorMessage)
    Write-Host "An error occurred: $ErrorMessage" -ForegroundColor Red
}

# Disable Startup Programs
try {
    Get-CimInstance Win32_StartupCommand | Where-Object { $_.Location -eq "HKLM" -or $_.Location -eq "HKCU" } | ForEach-Object { 
        $_ | Remove-CimInstance -ErrorAction Stop 
    } -ErrorAction Stop
    Write-Host "Startup programs disabled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to disable startup programs."
}

# Set High-Performance Power Plan
try {
    Start-Process "powercfg" -ArgumentList "-setactive SCHEME_MAX" -Wait -NoNewWindow -ErrorAction Stop
    Write-Host "High-Performance power plan set successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to set High-Performance power plan."
}

# Perform Disk Cleanup
try {
    Start-Process cleanmgr -ArgumentList "/sagerun:1" -Verb RunAs -Wait -ErrorAction Stop
    Write-Host "Disk cleanup executed successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to perform disk cleanup."
}

# Disable Unnecessary Windows Services (SysMain and Windows Search)
try {
    Get-Service | Where-Object { $_.DisplayName -like "Windows Search" -or $_.DisplayName -like "SysMain" } | ForEach-Object {
        Stop-Service -Name $_.Name -Force -ErrorAction SilentlyContinue
        Set-Service -Name $_.Name -StartupType Disabled -ErrorAction Stop
    } -ErrorAction Stop
    Write-Host "Unnecessary services disabled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to disable unnecessary services."
}

# Adjust Virtual Memory
try {
    $cs = Get-WmiObject Win32_ComputerSystem
    $cs.AutomaticManagedPagefile = $False
    $cs.Put()
    Set-WmiInstance -Class Win32_PageFileSetting -Arguments @{Name = "C:\pagefile.sys"; InitialSize = 4096; MaximumSize = 8192} -ErrorAction Stop
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
    Get-AppxPackage | Where-Object { $_.Name -like "*bing*" -or $_.Name -like "*games*" } | ForEach-Object {
        Remove-AppxPackage -Package $_ -ErrorAction SilentlyContinue
    }
    Write-Host "Bloatware uninstalled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to uninstall bloatware."
}

# Disable Animations for Speed
try {
    Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "VisualFXSetting" -Value 2 -ErrorAction Stop
    Write-Host "Animations disabled successfully." -ForegroundColor Green
} catch {
    Handle-Error "Failed to disable animations."
}

# Display completion message
[Microsoft.VisualBasic.Interaction]::MsgBox("All Done. Thanks Dani.", "OKOnly", "Optimization Script")
