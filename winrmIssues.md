# Enable WinRM
Enable-PSRemoting -Force

# Start the WinRM service
Start-Service winrm

# Set the WinRM service to automatically start
Set-Service winrm -StartupType Automatic


in CMD 
winrm quickconfig




http


# Enables the WinRM service and sets up the HTTP listener
Enable-PSRemoting -Force

# Opens port 5985 for all profiles
$firewallParams = @{
    Action      = 'Allow'
    Description = 'Inbound rule for Windows Remote Management via WS-Management. [TCP 5985]'
    Direction   = 'Inbound'
    DisplayName = 'Windows Remote Management (HTTP-In)'
    LocalPort   = 5985
    Profile     = 'Any'
    Protocol    = 'TCP'
}
New-NetFirewallRule @firewallParams

# Allows local user accounts to be used with WinRM
# This can be ignored if using domain accounts
$tokenFilterParams = @{
    Path         = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
    Name         = 'LocalAccountTokenFilterPolicy'
    Value        = 1
    PropertyType = 'DWORD'
    Force        = $true
}
New-ItemProperty @tokenFilterParams


# Allow HTTP (port 5985) through the firewall
New-NetFirewallRule -Name "Allow WinRM HTTP" -DisplayName "Allow WinRM HTTP" -Enabled True -Protocol TCP -Direction Inbound -Action Allow -LocalPort 5985

# Allow HTTPS (port 5986) through the firewall
New-NetFirewallRule -Name "Allow WinRM HTTPS" -DisplayName "Allow WinRM HTTPS" -Enabled True -Protocol TCP -Direction Inbound -Action Allow -LocalPort 5986


#Creating new certificte and addin
New-SelfSignedCertificate -DnsName "192.168.112.129" -CertStoreLocation "cert:\LocalMachine\My"


$certThumbprint = (Get-ChildItem -Path cert:\LocalMachine\My | Where-Object {$_.Subject -like "*192.168.112.129*"}).Thumbprint
$certThumbprint.GetType()
winrm create winrm/config/service/CertificateThumbprint $certThumbprint

Set-NetConnectionProfile -Name "Network 2" -NetworkCategory Private


# Enable WinRM
Enable-PSRemoting -Force

# Start the WinRM service
Start-Service winrm

# Set the WinRM service to automatically start
Set-Service winrm -StartupType Automatic

# Enable HTTPS listener
winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname="your.hostname";CertificateThumbprint="your_cert_thumbprint"}

Remove-Item -Path "cert:\LocalMachine\My\7E0AB98ABF8D1821DF03A9B4B298CDE477F197E8"


#cmd based commands
winrm quickconfig
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'




#Adding New firewall rule

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy" -Value 1
Restart-Service winrm

New-NetFirewallRule -DisplayName "WinRM (HTTP)" -Name "WinRM_HTTP" -Protocol TCP -LocalPort 5985 -Action Allow
New-NetFirewallRule -DisplayName "WinRM (HTTPS)" -Name "WinRM_HTTPS" -Protocol TCP -LocalPort 5986 -Action Allow


# Enables the WinRM service and sets up the HTTP listener
Enable-PSRemoting -Force

# Opens port 5985 for all profiles
$firewallParams = @{
    Action      = 'Allow'
    Description = 'Inbound rule for Windows Remote Management via WS-Management. [TCP 5985]'
    Direction   = 'Inbound'
    DisplayName = 'Windows Remote Management (HTTP-In)'
    LocalPort   = 5985
    Profile     = 'Any'
    Protocol    = 'TCP'
}
New-NetFirewallRule @firewallParams

# Allows local user accounts to be used with WinRM
# This can be ignored if using domain accounts
$tokenFilterParams = @{
    Path         = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
    Name         = 'LocalAccountTokenFilterPolicy'
    Value        = 1
    PropertyType = 'DWORD'
    Force        = $true
}
New-ItemProperty @tokenFilterParams



# Enables the WinRM service and sets up the HTTP listener
Enable-PSRemoting -Force

# Opens port 5985 for all profiles
$firewallParams = @{
    Action      = 'Allow'
    Description = 'Inbound rule for Windows Remote Management via WS-Management. [TCP 5985]'
    Direction   = 'Inbound'
    DisplayName = 'Windows Remote Management (HTTP-In)'
    LocalPort   = 5985
    Profile     = 'Any'
    Protocol    = 'TCP'
}
New-NetFirewallRule @firewallParams

# Allows local user accounts to be used with WinRM
# This can be ignored if using domain accounts
$tokenFilterParams = @{
    Path         = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
    Name         = 'LocalAccountTokenFilterPolicy'
    Value        = 1
    PropertyType = 'DWORD'
    Force        = $true
}
New-ItemProperty @tokenFilterParams





#Configremoteforansible.ps1
# Configure a Windows host for remote management with Ansible
# -----------------------------------------------------------
#
# This script checks the current WinRM/PSRemoting configuration and makes the
# necessary changes to allow Ansible to connect, authenticate and execute
# PowerShell commands.
#
# Set $VerbosePreference = "Continue" before running the script in order to
# see the output messages.
# Set $SkipNetworkProfileCheck to skip the network profile check.  Without
# specifying this the script will only run if the device's interfaces are in
# DOMAIN or PRIVATE zones.  Provide this switch if you want to enable winrm on
# a device with an interface in PUBLIC zone.
#
# Set $ForceNewSSLCert if the system has been syspreped and a new SSL Cert
# must be forced on the WinRM Listener when re-running this script. This
# is necessary when a new SID and CN name is created.
#
# Written by Trond Hindenes <trond@hindenes.com>
# Updated by Chris Church <cchurch@ansible.com>
# Updated by Michael Crilly <mike@autologic.cm>
# Updated by Anton Ouzounov <Anton.Ouzounov@careerbuilder.com>
# Updated by Nicolas Simond <contact@nicolas-simond.com>
#
# Version 1.0 - July 6th, 2014
# Version 1.1 - November 11th, 2014
# Version 1.2 - May 15th, 2015
# Version 1.3 - April 4th, 2016
# Version 1.4 - September 3rd, 2016

Param (
    [string]$SubjectName = $env:COMPUTERNAME,
    [int]$CertValidityDays = 1095,
    [switch]$SkipNetworkProfileCheck,
    $CreateSelfSignedCert = $true,
    [switch]$ForceNewSSLCert
)

Function New-LegacySelfSignedCert
{
    Param (
        [string]$SubjectName,
        [int]$ValidDays = 1095
    )

    $name = New-Object -COM "X509Enrollment.CX500DistinguishedName.1"
    $name.Encode("CN=$SubjectName", 0)

    $key = New-Object -COM "X509Enrollment.CX509PrivateKey.1"
    $key.ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
    $key.KeySpec = 1
    $key.Length = 4096
    $key.SecurityDescriptor = "D:PAI(A;;0xd01f01ff;;;SY)(A;;0xd01f01ff;;;BA)(A;;0x80120089;;;NS)"
    $key.MachineContext = 1
    $key.Create()

    $serverauthoid = New-Object -COM "X509Enrollment.CObjectId.1"
    $serverauthoid.InitializeFromValue("1.3.6.1.5.5.7.3.1")
    $ekuoids = New-Object -COM "X509Enrollment.CObjectIds.1"
    $ekuoids.Add($serverauthoid)
    $ekuext = New-Object -COM "X509Enrollment.CX509ExtensionEnhancedKeyUsage.1"
    $ekuext.InitializeEncode($ekuoids)

    $cert = New-Object -COM "X509Enrollment.CX509CertificateRequestCertificate.1"
    $cert.InitializeFromPrivateKey(2, $key, "")
    $cert.Subject = $name
    $cert.Issuer = $cert.Subject
    $cert.NotBefore = (Get-Date).AddDays(-1)
    $cert.NotAfter = $cert.NotBefore.AddDays($ValidDays)
    $cert.X509Extensions.Add($ekuext)
    $cert.Encode()

    $enrollment = New-Object -COM "X509Enrollment.CX509Enrollment.1"
    $enrollment.InitializeFromRequest($cert)
    $certdata = $enrollment.CreateRequest(0)
    $enrollment.InstallResponse(2, $certdata, 0, "")

    # Return the thumbprint of the last installed certificate;
    # This is needed for the new HTTPS WinRM listerner we're
    # going to create further down.
    Get-ChildItem "Cert:\LocalMachine\my"| Sort-Object NotBefore -Descending | Select -First 1 | Select -Expand Thumbprint
}

# Setup error handling.
Trap
{
    $_
    Exit 1
}
$ErrorActionPreference = "Stop"

# Detect PowerShell version.
If ($PSVersionTable.PSVersion.Major -lt 3)
{
    Throw "PowerShell version 3 or higher is required."
}

# Find and start the WinRM service.
Write-Verbose "Verifying WinRM service."
If (!(Get-Service "WinRM"))
{
    Throw "Unable to find the WinRM service."
}
ElseIf ((Get-Service "WinRM").Status -ne "Running")
{
    Write-Verbose "Starting WinRM service."
    Start-Service -Name "WinRM" -ErrorAction Stop
}

# WinRM should be running; check that we have a PS session config.
If (!(Get-PSSessionConfiguration -Verbose:$false) -or (!(Get-ChildItem WSMan:\localhost\Listener)))
{
  if ($SkipNetworkProfileCheck) {
    Write-Verbose "Enabling PS Remoting without checking Network profile."
    Enable-PSRemoting -SkipNetworkProfileCheck -Force -ErrorAction Stop
  }
  else {
    Write-Verbose "Enabling PS Remoting"
    Enable-PSRemoting -Force -ErrorAction Stop
  }
}
Else
{
    Write-Verbose "PS Remoting is already enabled."
}

# Make sure there is a SSL listener.
$listeners = Get-ChildItem WSMan:\localhost\Listener
If (!($listeners | Where {$_.Keys -like "TRANSPORT=HTTPS"}))
{
    # HTTPS-based endpoint does not exist.
    If (Get-Command "New-SelfSignedCertificate" -ErrorAction SilentlyContinue)
    {
        $cert = New-SelfSignedCertificate -DnsName $SubjectName -CertStoreLocation "Cert:\LocalMachine\My"
        $thumbprint = $cert.Thumbprint
        Write-Host "Self-signed SSL certificate generated; thumbprint: $thumbprint"
    }
    Else
    {
        $thumbprint = New-LegacySelfSignedCert -SubjectName $SubjectName
        Write-Host "(Legacy) Self-signed SSL certificate generated; thumbprint: $thumbprint"
    }

    # Create the hashtables of settings to be used.
    $valueset = @{}
    $valueset.Add('Hostname', $SubjectName)
    $valueset.Add('CertificateThumbprint', $thumbprint)

    $selectorset = @{}
    $selectorset.Add('Transport', 'HTTPS')
    $selectorset.Add('Address', '*')

    Write-Verbose "Enabling SSL listener."
    New-WSManInstance -ResourceURI 'winrm/config/Listener' -SelectorSet $selectorset -ValueSet $valueset
}
Else
{
    Write-Verbose "SSL listener is already active."
    
    # Force a new SSL cert on Listener if the $ForceNewSSLCert
    if($ForceNewSSLCert){
        
        # Create the new cert.
        If (Get-Command "New-SelfSignedCertificate" -ErrorAction SilentlyContinue)
        {
            $cert = New-SelfSignedCertificate -DnsName $SubjectName -CertStoreLocation "Cert:\LocalMachine\My"
            $thumbprint = $cert.Thumbprint
            Write-Host "Self-signed SSL certificate generated; thumbprint: $thumbprint"
        }
        Else
        {
            $thumbprint = New-LegacySelfSignedCert -SubjectName $SubjectName
            Write-Host "(Legacy) Self-signed SSL certificate generated; thumbprint: $thumbprint"
        }

        $valueset = @{}
        $valueset.Add('Hostname', $SubjectName)
        $valueset.Add('CertificateThumbprint', $thumbprint)

        # Delete the listener for SSL
        $selectorset = @{}
        $selectorset.Add('Transport', 'HTTPS')
        $selectorset.Add('Address', '*')
        Remove-WSManInstance -ResourceURI 'winrm/config/Listener' -SelectorSet $selectorset

        # Add new Listener with new SSL cert
        New-WSManInstance -ResourceURI 'winrm/config/Listener' -SelectorSet $selectorset -ValueSet $valueset
    }
}

# Check for basic authentication.
$basicAuthSetting = Get-ChildItem WSMan:\localhost\Service\Auth | Where {$_.Name -eq "Basic"}
If (($basicAuthSetting.Value) -eq $false)
{
    Write-Verbose "Enabling basic auth support."
    Set-Item -Path "WSMan:\localhost\Service\Auth\Basic" -Value $true
}
Else
{
    Write-Verbose "Basic auth is already enabled."
}

# Configure firewall to allow WinRM HTTPS connections.
$fwtest1 = netsh advfirewall firewall show rule name="Allow WinRM HTTPS"
$fwtest2 = netsh advfirewall firewall show rule name="Allow WinRM HTTPS" profile=any
If ($fwtest1.count -lt 5)
{
    Write-Verbose "Adding firewall rule to allow WinRM HTTPS."
    netsh advfirewall firewall add rule profile=any name="Allow WinRM HTTPS" dir=in localport=5986 protocol=TCP action=allow
}
ElseIf (($fwtest1.count -ge 5) -and ($fwtest2.count -lt 5))
{
    Write-Verbose "Updating firewall rule to allow WinRM HTTPS for any profile."
    netsh advfirewall firewall set rule name="Allow WinRM HTTPS" new profile=any
}
Else
{
    Write-Verbose "Firewall rule already exists to allow WinRM HTTPS."
}

# Test a remoting connection to localhost, which should work.
$httpResult = Invoke-Command -ComputerName "localhost" -ScriptBlock {$env:COMPUTERNAME} -ErrorVariable httpError -ErrorAction SilentlyContinue
$httpsOptions = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck

$httpsResult = New-PSSession -UseSSL -ComputerName "localhost" -SessionOption $httpsOptions -ErrorVariable httpsError -ErrorAction SilentlyContinue

If ($httpResult -and $httpsResult)
{
    Write-Verbose "HTTP: Enabled | HTTPS: Enabled"
}
ElseIf ($httpsResult -and !$httpResult)
{
    Write-Verbose "HTTP: Disabled | HTTPS: Enabled"
}
ElseIf ($httpResult -and !$httpsResult)
{
    Write-Verbose "HTTP: Enabled | HTTPS: Disabled"
}
Else
{
    Throw "Unable to establish an HTTP or HTTPS remoting session."
}
Write-Verbose "PS Remoting has been successfully configured for Ansible."

