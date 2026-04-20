# Multi-Site-Enterprise-Network-Deployment
This lab demonstrates the deployment of a full Windows Server 2025 infrastructure for a distributed corporation (Progreso Corp).

Key Implementations: 
Active Directory Domain Services (AD DS): Forest creation and multi-DC promotion using PowerShell.
Network Services: Configuration of centralized DNS and multi-scope DHCP for branch offices (Halifax & Mexico).
Routing: Implementation of RRAS (Routing and Remote Access) to enable inter-VLAN communication.
Topology:

<img width="771" height="575" alt="image" src="https://github.com/user-attachments/assets/792c23ac-3419-4a05-a52c-e94146cfd93d" />


1. Automated Active Directory Deployment
This script handles the DC promotion for EDM-SVR1.

PowerShell
# Install AD DS Binaries
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promotion Script for Secondary Domain Controller
$domain = "progreso.com"
$cred = Get-Credential # Use PROGRESO\Administrator

Install-ADDSDomainController `
    -NoGlobalCatalog:$true `
    -Credential $cred `
    -DomainName $domain `
    -DatabasePath "C:\Windows\NTDS" `
    -LogPath "C:\Windows\NTDS" `
    -SysvolPath "C:\Windows\SYSVOL" `
    -Force:$true
2. Multi-Scope DHCP Configuration
This script automates the creation of the Halifax and Mexico scopes from Assignment 4.

PowerShell
# Install DHCP Role
Install-WindowsFeature DHCP -IncludeManagementTools

# Authorize DHCP Server in AD
Add-DhcpServerInDC -DnsName "EDM-SVR1.progreso.com"

# Create Halifax Scope (172.16.30.0/24)
Add-DhcpServerv4Scope -Name "Halifax-Branch" -StartRange 172.16.30.100 -EndRange 172.16.30.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -ScopeId 172.16.30.0 -Router 172.16.30.1 -DnsServer 192.168.0.2

# Create Mexico Scope (172.16.35.0/24)
Add-DhcpServerv4Scope -Name "Mexico-Branch" -StartRange 172.16.35.100 -EndRange 172.16.35.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -ScopeId 172.16.35.0 -Router 172.16.35.1 -DnsServer 192.168.0.2
3. Network Health Check (Post-Deployment Audit)
Include this to show you know how to verify your work.

PowerShell
Write-Host "Verifying AD Replication..." -ForegroundColor Green
repadmin /showrepl

Write-Host "Verifying DNS Resolution..." -ForegroundColor Green
Resolve-DnsName progreso.com

Write-Host "Checking DHCP Scopes..." -ForegroundColor Green
Get-DhcpServerv4Scope
