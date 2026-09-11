# Experiment 6: Memory Forensics Using Volatility 3

## AIM

To perform RAM forensics on the **`lasten.raw` Windows memory image** using **Volatility 3** and identify suspicious processes, files, PowerShell activity, persistence mechanisms, and network indicators.

## Tools Required

- Kali Linux
- Python 3
- Volatility 3
- Windows memory image: `lasten.raw` [Download/View lasten Memory Image](https://drive.google.com/file/d/1hlxiV4N9y_7RvjyjkBtUdm2ZCUQUcXYK/view?usp=sharing)
- Terminal

## Objectives

1. To identify the operating system from the memory image.
2. To analyze running and terminated processes.
3. To examine the process tree and command-line arguments.
4. To identify suspicious processes and files.
5. To analyze network-related artifacts.
6. To check suspicious memory regions.
7. To search memory for malicious or suspicious strings.
8. To identify persistence and reverse-shell indicators.

## Theory

**Memory forensics** is the process of analyzing the contents of RAM to obtain information about a computer system at a particular point in time.

**Volatility 3** is an open-source memory-forensics framework used to analyze memory dumps. It can provide information about processes, network connections, files, command lines, and other artifacts present in memory.

---

# Procedure

## Step 1: Install and Set Up Volatility 3

First, update the package list and install the required Python tools.

### Commands

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv git -y
```

Clone Volatility 3:

```bash
cd ~/Desktop/Exp6
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
```

Create and activate a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

Install Volatility 3:

```bash
pip install -e .
```

### Observation

The Volatility 3 framework and Python dependencies were installed successfully.

### Result

The Volatility 3 environment was prepared for memory analysis.

---

## Step 2: Check Volatility Version

Verify that Volatility 3 is working correctly.

### Command

```bash
python3 vol.py --version
```

### Observation

The installed Volatility 3 version was displayed. In this analysis, version **2.28.2** was used.

### Result

Volatility 3 was successfully installed and ready for memory analysis.

---

## Step 3: Verify the Memory Image

Make sure the memory image is available in the Volatility directory.

### Commands

```bash
ls -lh lasten.raw
```

```bash
file lasten.raw
```

### Observation

The `lasten.raw` memory image was present and available for analysis.

### Result

The memory image was successfully located and prepared for forensic analysis.

---

## Step 4: Identify the Operating System

The `windows.info` plugin was used to identify the operating-system information.

### Command

```bash
python3 vol.py -f lasten.raw windows.info
```

### Observation

The memory image was identified as a **Windows 7 SP1 64-bit** system.

Important information included:

- Windows version: Windows 7 SP1
- Architecture: 64-bit
- Kernel information was successfully detected.

### Result

The operating system of the memory image was successfully identified as Windows 7 SP1 64-bit.

---

## Step 5: List Running Processes

The `windows.pslist` plugin was used to list processes present in memory.

### Command

```bash
python3 vol.py -f lasten.raw windows.pslist
```

### Observation

Several processes were identified. Some important entries included:

- `FIFA23.exe`
- `notepad.exe`
- `wampmanager.exe`
- `mysqld.exe`
- `DumpIt.exe`
- `dllhost.exe`

A `notepad.exe` process was associated with:

```text
C:\Users\vboxuser\AppData\Local\backagainn.ps1
```

### Result

The active process list was successfully examined and suspicious artifacts were selected for further investigation.

---

## Step 6: Scan for Terminated or Unlinked Processes

The `windows.psscan` plugin was used because a process may no longer appear in the normal active process list.

### Command

```bash
python3 vol.py -f lasten.raw windows.psscan
```

### Observation

A process named **`notapadd.exe`** was identified:

- PID: **2392**
- Created: **2023-11-17 15:35:16 UTC**
- Exited: **2023-11-17 15:35:38 UTC**

The process appeared in `psscan` but not in the active `pslist` output.

### Result

`notapadd.exe` was identified as a terminated process artifact and selected for further investigation.

---

## Step 7: Analyze the Process Tree

The `windows.pstree` plugin was used to examine parent-child process relationships.

### Command

```bash
python3 vol.py -f lasten.raw windows.pstree
```

### Observation

The process hierarchy showed normal Windows processes along with applications such as `FIFA23.exe`, `notepad.exe`, WAMP processes, and other processes.

`notapadd.exe` was not present in the current process tree because it had already terminated.

### Result

Process relationships were successfully analyzed.

---

## Step 8: Examine Process Command Lines

The `windows.cmdline` plugin was used to identify command-line arguments and execution paths.

### Command

```bash
python3 vol.py -f lasten.raw windows.cmdline
```

### Observation

An important entry was:

```text
"C:\Windows\System32\notepad.exe" "C:\Users\vboxuser\AppData\Local\backagainn.ps1"
```

Another suspicious application was:

```text
C:\Users\vboxuser\Downloads\FIFA23.exe
```

### Result

The command-line analysis provided important clues about files and scripts that were present or accessed during the memory capture.

---

## Step 9: Analyze Network Connections

The `windows.netscan` plugin was used to examine network connections and listening sockets.

### Command

```bash
python3 vol.py -f lasten.raw windows.netscan
```

### Observation

Several Windows services and applications had network-related artifacts.

`mysqld.exe` was observed listening on port **3307**.

No active connection to **149.100.50.25:4444** was directly confirmed in the `netscan` output.

### Result

Network artifacts were successfully identified and the suspicious IP/port was investigated further using memory strings.

---

## Step 10: Scan Files in Memory

The `windows.filescan` plugin was used to find file objects present in memory.

### Command

```bash
python3 vol.py -f lasten.raw windows.filescan
```

### Observation

Important file artifacts included:

```text
C:\Users\vboxuser\Desktop\notapadd.exe
C:\Users\vboxuser\AppData\Local\backagainn.ps1
C:\Users\vboxuser\AppData\Local\Createback.ps1
C:\Users\vboxuser\Desktop\InfoBank.txt
C:\Users\vboxuser\Downloads\FIFA23.exe
```

Windows Error Reporting entries related to `notapadd.exe` were also present.

### Result

Suspicious executable and PowerShell script artifacts were identified in memory.

---

## Step 11: Analyze Suspicious Memory Using Malfind

The `windows.malfind` plugin was used to check for potentially injected or suspicious executable memory regions.

### Command

```bash
python3 vol.py -f lasten.raw windows.malfind --pid 2392
```

### Observation

No findings were reported for PID **2392**.

### Result

No evidence of process injection was identified for `notapadd.exe` using `malfind`.

---

## Step 12: Check the Suspicious Process Command Line

The command line of `notapadd.exe` was checked separately.

### Command

```bash
python3 vol.py -f lasten.raw windows.cmdline --pid 2392
```

### Observation

No command-line information was recovered for PID 2392.

### Result

The command-line information for the terminated process could not be recovered.

---

## Step 13: Attempt to Dump Cached Files

The `windows.dumpfiles` plugin was used to attempt recovery of cached file contents.

### Command

```bash
python3 vol.py -f lasten.raw windows.dumpfiles
```

### Observation

Volatility attempted to recover cached file objects, but the required contents of `backagainn.ps1` were not successfully recovered.

### Result

Direct recovery of the required cached script contents was unsuccessful.

---

## Step 14: Search for Suspicious File Names

The Linux `strings` command was used to search readable strings in the raw memory image.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'notapadd|backagainn|FIFA23|InfoBank|Bank_Info'
```

### Observation

References to the following artifacts were found:

- `notapadd.exe`
- `backagainn.ps1`
- `FIFA23.exe`
- `InfoBank.txt`
- `Bank_Info.txt`

### Result

Multiple suspicious file artifacts were confirmed in the memory image.

---

## Step 15: Search for PowerShell Activity

PowerShell-related strings were searched in the memory image.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'powershell|IEX|Invoke-Expression|New-Object|schtasks'
```

### Observation

Important strings included:

```text
powershell
New-Object
Invoke-Expression
schtasks.exe
```

### Result

The memory image contained evidence of PowerShell-based activity.

---

## Step 16: Search for Scheduled Task Persistence

Scheduled-task related strings were searched to investigate persistence.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'ScheduledTask|schtasks|Backagainn'
```

### Observation

The memory image contained strings such as:

```text
schtasks.exe /create /tn "defender monitor"
```

and PowerShell code containing:

```text
New-ScheduledTask
TaskName "Backagainn"
Register-ScheduledTask
```

### Result

Evidence of scheduled-task based persistence was identified in memory.

---

## Step 17: Search for Suspicious IP Address and Port

The suspicious IP address and port were searched in the raw memory.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei '149\.100\.50\.25|4444'
```

### Observation

The following network indicators were found:

```text
149.100.50.25
4444
```

### Result

The memory image contained an artifact referencing communication with **149.100.50.25 on port 4444**.

---

## Step 18: Search for Reverse-Shell Indicators

Reverse-shell related strings were searched.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'TCPClient|NetworkStream|StreamReader|StreamWriter|Invoke-Expression'
```

### Observation

The following PowerShell components were found:

```text
Net.Sockets.TCPClient
NetworkStream
StreamReader
StreamWriter
Invoke-Expression
```

A PowerShell script artifact also contained the target:

```text
$LHOST = "149.100.50.25"
$LPORT = 4444
```

### Result

A PowerShell reverse-shell script artifact was identified in the memory image.

---

## Step 19: Investigate `backagainn.ps1`

The suspicious PowerShell script was searched with surrounding strings.

### Command

```bash
strings -a -n 6 lasten.raw | grep -i -A 30 -B 10 'backagainn.ps1'
```

### Observation

References to the script were found, including:

```text
C:\Users\vboxuser\AppData\Local\backagainn.ps1
```

The memory also contained references showing that the script had been accessed/opened.

### Result

`backagainn.ps1` was confirmed as an important memory-resident artifact.

---

## Step 20: Investigate `notapadd.exe`

References to the suspicious executable were searched.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'notapadd.exe'
```

### Observation

The following path was found:

```text
C:\Users\vboxuser\Desktop\notapadd.exe
```

### Result

The `notapadd.exe` executable was confirmed as an artifact present in the memory image.

---

## Step 21: Search the PowerShell Script Path

The PowerShell script path was searched directly.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'backagainn.ps1'
```

### Observation

Multiple references to:

```text
C:\Users\vboxuser\AppData\Local\backagainn.ps1
```

were found.

### Result

The presence of the suspicious PowerShell script was confirmed.

---

## Step 22: Perform Final IOC Search

A combined search was performed to correlate the major indicators.

### Command

```bash
strings -a -n 8 lasten.raw | grep -Ei 'notapadd|backagainn|149\.100\.50\.25|4444|schtasks|powershell'
```

### Observation

Multiple related indicators were found:

- `notapadd.exe`
- `backagainn.ps1`
- `powershell`
- `schtasks`
- `Backagainn`
- `149.100.50.25`
- `4444`

Additional reverse-shell indicators included:

- `TCPClient`
- `NetworkStream`
- `Invoke-Expression`

### Result

The final IOC search correlated the suspicious executable, PowerShell activity, scheduled-task persistence, and reverse-shell script artifacts.

---

# Overall Result

The `lasten.raw` Windows 7 SP1 64-bit memory image was successfully analyzed using Volatility 3.

The investigation identified:

1. A terminated process named **`notapadd.exe` (PID 2392)**.
2. References to **`backagainn.ps1`** and **`Createback.ps1`**.
3. PowerShell-related activity.
4. Scheduled-task persistence indicators involving **`Backagainn`**.
5. A suspicious PowerShell reverse-shell artifact referencing **`149.100.50.25:4444`**.
6. Use of `Net.Sockets.TCPClient`, `NetworkStream`, and `Invoke-Expression`.
7. No `malfind` evidence of process injection for PID 2392.
8. No confirmed active connection to `149.100.50.25:4444` in the `netscan` output at the time represented by the memory image.

## Conclusion

Volatility 3 was used to investigate the Windows memory image through process analysis, process scanning, process-tree analysis, command-line analysis, network analysis, file scanning, memory analysis, and string-based IOC searching.

The combined evidence indicates **strong signs of malicious PowerShell activity and persistence**, including a scheduled task and reverse-shell code. The reverse-shell code was recovered as a memory/script artifact; therefore, it should not be stated that the reverse shell was definitely active during the memory capture.

The experiment demonstrates how memory forensics can be used to recover and correlate evidence from a suspicious Windows system without relying only on files stored on disk.
