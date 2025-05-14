# Configure WinRM on target host

This section guides you through the initial process of configuring WinRM and adding a user that Ansible playbooks will operate with on the windows host you want to automate. It is the first step when automating Windows hosts with Ansible!

!!! tip
    Depending on your demo environment or target host this might not be necessary, as WinRM is already configured.

## Preparation

Open a powershell terminal (as administrator) on the target host. Check, if powershell scripts can be executed:

```powershell
Get-ExecutionPolicy
```

If you get the message `Restricted`, you need to change the execution policy. To allow the execution of scripts permanently for the  user currently logged in execute the following command:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

If you run the `Get-ExecutionPolicy` command again, the output should now see `RemoteSigned`.

## Step 1 - Set up WinRM

Now that you can execute powershell scripts, let us create a script that sets up the Windows Remote Management Protocol `WinRM`. Try to find the script you need to configure WinRM in the [Ansible documentation](https://docs.ansible.com/ansible/latest/os_guide/intro_windows.html){:target="_blank"}. Then paste it into a text editor like notepad and save it as (for example) `winrm_script.ps1`.

??? example "You also can find the script you need here:"

    ```powershell
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
    ```

In the powershell terminal, change to the directory you saved the powershell script in (using `cd` and the path to the directory) and simply run:

```powershell
./winrm_script.ps1
```

Finally, confirm that the WinRM service is running and that port 5985 is open with the following command:

```powershell
winrm enumerate winrm/config/Listener
```

## Step 2 - Create a user for the execution of ansible playbooks

In the next step, we will create a dedicated user account on our Windows host that will be used to execute the Ansible playbooks designed for automating administrative tasks. This user account is necessary not only for authentication, but also for enforcing access control, maintaining security, and ensuring consistent automation behavior. For example, we can assign this user only the specific permissions required for automation, without granting full administrative rights.

To proceed, open an elevated PowerShell terminal and create a new user named `ansible` with the password `automation123`:

```powershell
net user ansible automation123 /add
```

To make it simple for now, we give our user administration rights by adding him to the corresponding group:

```powershell
net localgroup Administrators ansible /add
```

Finally, let us confirm that the user is created and in the admin group:

```powershell
Get-LocalUser -Name “ansible”
Get-LocalGroupMember -Group "Administrators"
```
