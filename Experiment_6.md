# Experiment 6: Memory Forensics Using Volatility 3

## AIM

To perform memory forensics on a Windows memory image (`lasten.raw`) using **Volatility 3** and identify useful forensic artifacts such as operating-system information, processes, files, command lines, network activity, and suspicious PowerShell activity.

---

## TOOLS REQUIRED

- Kali Linux
- Python 3
- Volatility 3
- Windows memory image: `lasten.raw` [Download lasten Memory Image](https://drive.google.com/file/d/1hlxiV4N9y_7RvjyjkBtUdm2ZCUQUcXYK/view?usp=sharing)
- Terminal
- Basic Linux commands such as `strings`, `grep`, and `file`

---

## OBJECTIVES

1. To identify the operating system from the memory image.
2. To list active and terminated processes.
3. To examine the process hierarchy and command lines.
4. To identify suspicious processes and files.
5. To examine network artifacts.
6. To search memory for suspicious PowerShell commands and persistence mechanisms.
7. To correlate multiple artifacts and identify possible indicators of compromise (IOCs).

---

## THEORY

**Memory forensics** is the process of examining the contents of RAM to recover information about a computer system at a particular point in time.

**Volatility 3** is an open-source memory-forensics framework. It can be used to investigate processes, process relationships, command lines, network connections, files, memory regions, and other artifacts present in a memory image.

In this experiment, the Windows memory image `lasten.raw` is analyzed using Volatility 3. The investigation is performed by first identifying the operating system, then examining processes and their relationships, followed by files, network artifacts, and suspicious strings.

---

# PROCEDURE

## Step 1: Install and Set Up Volatility 3

### Command

```bash
cd ~/Desktop/Exp6
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3

python3 -m venv venv
source venv/bin/activate

pip install -e .
```

If Volatility 3 is already installed, only activate the existing environment:

```bash
cd ~/Desktop/Exp6/volatility3
source venv/bin/activate
```

### Observation

The Volatility 3 environment was created and the framework was installed.

### Result

Volatility 3 was successfully prepared for memory analysis.

---

## Step 2: Check the Volatility Version

### Command

```bash
python3 vol.py --version
```

### Observation

The installed framework version was displayed.

### Result

Volatility 3 Framework **2.28.2** was available for the experiment.

---

## Step 3: Identify the Operating System

The `windows.info` plugin was used first to determine the operating-system details of the memory image.

### Command

```bash
python3 vol.py -f lasten.raw windows.info
```

### Observation

The output identified the memory image as:

- Windows 7
- Service Pack 1
- 64-bit system
- Build information: `7601.17514.amd64fre.win7sp1_rtm`
- System time: `2023-11-18 00:19:37 UTC`

### Result

The memory image was identified as a **Windows 7 SP1 64-bit** memory image.

---

## Step 4: List Active Processes

The `windows.pslist` plugin was used to list processes present in the normal active process list.

### Command

```bash
python3 vol.py -f lasten.raw windows.pslist
```

### Observation

Several normal Windows processes were present. Some user-level processes were also observed, including:

- `FIFA23.exe` — PID **1496**
- `notepad.exe` — PID **3596**, with command line referring to:
  `C:\Users\vboxuser\AppData\Local\backagainn.ps1`
- `notepad.exe` — PID **1676**, opening:
  `C:\Users\vboxuser\Desktop\InfoBank.txt`
- `notepad.exe` — PID **3688**, opening:
  `C:\Users\vboxuser\Desktop\Bank_Info.txt`

An unusual `dllhost.exe` process with PID **632** was also observed with an extremely large handle count.

### Result

The active process list was obtained. User-level processes and unusual process characteristics were identified for further investigation.

---

## Step 5: Scan Memory for Process Objects

`windows.psscan` was used because a process may no longer appear in the normal process list after termination.

### Command

```bash
python3 vol.py -f lasten.raw windows.psscan
```

### Observation

A process named:

```text
notapadd.exe
PID: 2392
PPID: 1072
```

was found by the memory scan.

Important details included:

- Create time: `2023-11-17 15:35:16 UTC`
- Exit time: `2023-11-17 15:35:38 UTC`
- The process was not present in the normal `pslist` output.

### Result

`notapadd.exe` was identified as a **terminated process artifact** in memory. Its presence required further investigation, but `psscan` alone does not prove that the process was malicious.

---

## Step 6: Examine the Process Tree

The `windows.pstree` plugin was used to understand parent-child process relationships.

### Command

```bash
python3 vol.py -f lasten.raw windows.pstree
```

### Observation

The process hierarchy showed normal Windows parent-child relationships. For example, `explorer.exe` (PID 1072) was associated with several user processes, including `FIFA23.exe`.

The terminated `notapadd.exe` process was not displayed in the process tree because it was no longer part of the active process hierarchy.

### Result

The process tree helped establish process relationships and distinguish active processes from the terminated process artifact found by `psscan`.

---

## Step 7: Examine Command-Line Arguments

The `windows.cmdline` plugin was used to determine how processes were launched.

### Command

```bash
python3 vol.py -f lasten.raw windows.cmdline
```

### Observation

A significant entry was:

```text
PID 3596
notepad.exe
"C:\Windows\System32\notepad.exe" "C:\Users\vboxuser\AppData\Local\backagainn.ps1"
```

This shows that `notepad.exe` had opened the file:

```text
C:\Users\vboxuser\AppData\Local\backagainn.ps1
```

Other command-line entries included `FIFA23.exe`, WAMP processes, and `DumpIt.exe`.

### Result

The command-line analysis provided a direct reference to `backagainn.ps1`, making it an important artifact for further investigation.

---

## Step 8: Check the Terminated Process Command Line

The command line of PID 2392 was checked separately.

### Command

```bash
python3 vol.py -f lasten.raw windows.cmdline --pid 2392
```

### Observation

Only the column headers were returned and no command-line entry was recovered for PID 2392.

### Result

A command line for the terminated `notapadd.exe` process could not be recovered from this plugin.

---

## Step 9: Analyze Network Artifacts

The `windows.netscan` plugin was used to examine network connections and listening sockets.

### Command

```bash
python3 vol.py -f lasten.raw windows.netscan
```

### Observation

Network artifacts included normal Windows listeners and services. Examples included:

- SMB-related listeners on ports 139 and 445
- LSASS-related listening activity
- MariaDB/MySQL listening on port **3307**

No confirmed active connection to:

```text
149.100.50.25:4444
```

was observed in the `netscan` output.

### Result

Network artifacts were recovered from memory. The reverse-shell IP and port were not confirmed as an active network connection at the time represented by the memory image.

---

## Step 10: Scan Memory for File Objects

The `windows.filescan` plugin was used to search memory for file objects related to suspicious artifacts.

### Command

```bash
python3 vol.py -f lasten.raw windows.filescan | grep -Ei 'backagainn|notapadd|FIFA23|InfoBank|Bank_Info|aa\.txt|\.ps1'
```

### Observation

Important file references included:

```text
\Users\vboxuser\Downloads\FIFA23.exe
\Users\vboxuser\Desktop\notapadd.exe
\Users\vboxuser\Desktop\InfoBank.txt
\Users\vboxuser\AppData\Local\Createback.ps1
\Users\vboxuser\AppData\Local\backagainn.ps1
```

Windows Error Reporting files referring to `notapadd.exe` were also present.

### Result

Multiple suspicious or investigation-relevant file artifacts were identified in memory.

---

## Step 11: Attempt to Recover the PowerShell Script

The file object for `backagainn.ps1` was located by `filescan`. Its cached contents were then checked using `dumpfiles`.

### Command

```bash
python3 vol.py -f lasten.raw windows.dumpfiles --virtaddr 0x7fa1cc80
```

### Observation

No file object was returned and no recovered file was produced from this virtual address.

### Result

The `backagainn.ps1` file reference was present in memory, but its contents could not be recovered using `dumpfiles` from the identified virtual address.

Therefore, the script contents should **not** be claimed to have been extracted directly using `dumpfiles`.

---

## Step 12: Search the Memory Image Using Strings

The Linux `strings` command was used to extract readable text from the raw memory image.

### Command

```bash
strings -a -n 8 lasten.raw | less
```

### Observation

Readable strings related to user files, PowerShell, Windows commands, and suspicious filenames were found.

### Result

The `strings` output provided additional artifacts for correlation with the Volatility results.

---

## Step 13: Search for PowerShell and Command-Execution Artifacts

A targeted search was performed for PowerShell and command-execution indicators.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'powershell|Invoke-WebRequest|Invoke-Expression|IEX|New-Object|Net\.TcpClient|DownloadString|Start-Process|ScheduledTask|schtasks|Set-ItemProperty' | head -200
```

### Observation

Important strings included:

```text
Microsoft.PowerShell.ConsoleHost
schtasks.exe /delete
schtasks.exe /create /tn "defender monitor"
```

A PowerShell reverse-shell script was also recovered from memory strings. It contained:

```text
$LHOST = "149.100.50.25"
$LPORT = 4444
New-Object Net.Sockets.TCPClient
Invoke-Expression
```

A scheduled-task PowerShell script was also present, including:

```text
New-ScheduledTaskAction
New-ScheduledTaskTrigger
New-ScheduledTask
Register-ScheduledTask
-TaskName "Backagainn"
```

### Result

The memory image contained strong evidence of **PowerShell-based command execution, scheduled-task persistence, and reverse-shell functionality**.

---

## Step 14: Correlate the Main Suspicious Indicators

The important filenames, IP address, port, and task name were searched together.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei '149\.100\.50\.25|4444|Backagainn|backagainn\.ps1|notapadd\.exe' | head -100
```

### Observation

The output contained references to:

```text
notapadd.exe
C:\Users\vboxuser\Desktop\notapadd.exe
backagainn.lnk
C:\Users\vboxuser\AppData\Local\backagainn.ps1
149.100.50.25
4444
Net.Sockets.TCPClient
Invoke-Expression
Backagainn
```

### Result

The artifacts were strongly correlated with a PowerShell-based suspicious activity chain. However, the available memory evidence does not by itself prove that `notapadd.exe` directly launched `backagainn.ps1`.

---

## Step 15: Search Specifically for the Suspicious Script

### Command

```bash
strings -a -n 6 lasten.raw | grep -i -A 30 -B 10 "backagainn.ps1"
```

### Observation

References to `backagainn.ps1` were found, including:

```text
C:\Users\vboxuser\AppData\Local\backagainn.ps1
```

and a command-line reference showing:

```text
"C:\Windows\System32\notepad.exe" "C:\Users\vboxuser\AppData\Local\backagainn.ps1"
```

Browser-history style strings also referenced the same file.

### Result

The presence and use/reference of `backagainn.ps1` were supported by multiple memory artifacts.

---

## Step 16: Check for Memory Injection in the Terminated Process

The `windows.malfind` plugin was used to check PID 2392 for suspicious memory regions.

### Command

```bash
python3 vol.py -f lasten.raw windows.malfind --pid 2392
```

### Observation

The command produced no findings for PID 2392. A deprecation warning was also displayed because the plugin interface has been renamed in the Volatility 3 framework.

### Result

No injected-memory finding was recovered for PID 2392 using `malfind`. Therefore, memory injection should not be claimed based on this analysis.

---

## Step 17: Final IOC Search

A final combined search was used to correlate the main indicators.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'notapadd\.exe|backagainn\.ps1|Backagainn|149\.100\.50\.25|4444|powershell|schtasks|TCPClient|Invoke-Expression'
```

### Observation

The search returned multiple related indicators:

- `notapadd.exe`
- `backagainn.ps1`
- `Backagainn`
- `149.100.50.25`
- `4444`
- PowerShell
- `schtasks`
- `Net.Sockets.TCPClient`
- `Invoke-Expression`

### Result

The final IOC search correlated the main artifacts discovered during the investigation and provided strong evidence of suspicious PowerShell activity and persistence-related artifacts.

---

# OBSERVATION SUMMARY

| Investigation | Important Observation |
|---|---|
| `windows.info` | Windows 7 SP1 64-bit |
| `windows.pslist` | `FIFA23.exe`, `backagainn.ps1` reference through Notepad, other user processes |
| `windows.psscan` | Terminated `notapadd.exe`, PID 2392 |
| `windows.pstree` | Process hierarchy and parent-child relationships |
| `windows.cmdline` | PID 3596 opened `backagainn.ps1` |
| `windows.netscan` | Network artifacts found; no confirmed active `149.100.50.25:4444` connection |
| `windows.filescan` | `notapadd.exe`, `backagainn.ps1`, `Createback.ps1`, `FIFA23.exe` references |
| `windows.dumpfiles` | `backagainn.ps1` contents not recovered |
| `strings` | PowerShell, scheduled-task, reverse-shell, and IOC strings |
| `windows.malfind` | No finding for PID 2392 |

---

# OVERALL RESULT

The `lasten.raw` Windows memory image was successfully analyzed using **Volatility 3 Framework 2.28.2**.

The image was identified as a **Windows 7 SP1 64-bit** system. Process analysis identified normal system processes as well as investigation-relevant artifacts such as `FIFA23.exe` and the terminated `notapadd.exe` process (PID 2392).

Further analysis identified the file:

```text
C:\Users\vboxuser\AppData\Local\backagainn.ps1
```

and memory strings containing PowerShell commands associated with:

- scheduled-task creation,
- the task name `Backagainn`,
- `Net.Sockets.TCPClient`,
- remote host `149.100.50.25`,
- port `4444`,
- and `Invoke-Expression`.

These artifacts provide strong evidence of suspicious PowerShell-based activity and persistence mechanisms in the memory image.

However, the investigation did **not** confirm an active connection to `149.100.50.25:4444`, did **not** recover the contents of `backagainn.ps1` using `dumpfiles`, and did **not** find a `malfind` result for PID 2392.

---

# CONCLUSION

This experiment demonstrated how **Volatility 3** can be used for Windows memory forensics. By combining Volatility plugins such as `windows.info`, `windows.pslist`, `windows.psscan`, `windows.pstree`, `windows.cmdline`, `windows.netscan`, `windows.filescan`, and `windows.malfind` with Linux commands such as `strings` and `grep`, important forensic artifacts were recovered from the `lasten.raw` memory image.

The investigation showed that memory forensics can reveal process artifacts, file references, command-line information, network artifacts, and suspicious PowerShell commands even when complete files or active processes are no longer available.
