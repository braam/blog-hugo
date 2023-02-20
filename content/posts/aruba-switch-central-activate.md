---
title: "Aruba Switch Central Activate"
date: 2022-07-19T11:44:13+02:00
draft: false
tags:
- Aruba
---

Most of the time when connecting switches to the Aruba Central platform there is no issue at all, the switch receives through DHCP an IP address and is able to communicate with the platform. But sometimes it's not working like it should and we get this covered here.

## What is needed?
First of all following is needed to be able to succesful connect to the Aruba Central:
1) IP address + subnetmask + Router IP
2) DNS server
3) Correct time settings --> NTP
4) Firewall policies to allow traffic to Aruba Central
5) Aruba Central Subscription License

### CLI commands
1) Check internet access
    ```
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# ping 8.8.8.8
    ```
2) Check DNS resolving
    ```
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# ping www.google.com
    ```
3) Check time settings
    ```
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# sntp unicast
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# sntp server priority 1 216.239.35.4
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# time daylight-time-rule western-europe
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# time timezone 60
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# timesync ntp
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# ntp enable
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# show ntp status
    NTP Status Information

    NTP Status             : Enabled         NTP Mode        : Broadcast       
    Synchronization Status : None Configured Peer Dispersion : 0.00000 sec     
    Stratum Number         : 16              Leap Direction  : 0               
    Reference Assoc ID     : 0               Clock Offset    : 0.00000 sec     
    Reference ID           : 0.0.0.0         Root Delay      : 0.00000 sec     
    Precision              : 2**-18          Root Dispersion : 0.00267 sec     
    NTP Up Time            : 0d 0h 2m        Time Resolution : 819 nsec        
    Drift                  : 0.00000 sec/sec 

    System Time            : Tue Jul 19 10:39:30 2022                
    Reference Time         : Mon Jan  1 01:00:00 1990                
    ```
4) Reboot safe-proof
    ```
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# wr mem
    ```
5) Activate Aruba Central with Force
    ```
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# activate provision force
    Aruba-2930F-48G-740W-PoEP-4SFPP(config)# show activate provision 

    Configuration and Status - Activate Provision Service

    Activate Provision Service   : Enabled
    Activate Server Address      : devices-v2.arubanetworks.com
    Activation Key               : ARRMEMNR
    Time Sync Status             : Time sync from NTP pool
    Activate DNS Lookup          : Success
    Proxy Server DNS Lookup      : NA
    Activate Connection Status   : Success
    Error Reason                 : NA
    ```

&nbsp;
### Firewall Policies
All communications is done through TCP 443.
You can get the full FQDN lists for destinations that should become reachable here: [https://www.arubanetworks.com/techdocs/central/latest/content/nms/device-mgmt/communication_ports.htm](https://www.arubanetworks.com/techdocs/central/latest/content/nms/device-mgmt/communication_ports.htm)






