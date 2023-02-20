---
title: "Mitel DHCP Vendor Class Windows Server"
date: 2022-07-15T14:43:45+02:00
draft: false
tags:
- Mitel
- DHCP
- Windows Server
---

Mitel/Aastra phones with DHCP options is very challenging I did found out some differences between phones and options you need to configure.


### VCI (Vendor Class Identifier) Method 1
According to some documentations I did found online, but forgot to bookmark it :( We can give different options per phone model. Create for each model a different VCI.
The syntax is like follow: "AastraIPPhone**XXX**) In this example I'm using the phone model **AastraIPPhone6920**:  
![VCI Win](/posts_images/mitel-windows-vci-01.png)

After that we do need to create **option 08** and **option 09** on this VCI.
1) Option 08  
    With **option 08** we let the phone know that it'll apply to them, so they will need to accept all other options that'll come.
    Important to note is, the type for **option 08** needs to be a **Byte Array**.
    The acutal data needs to be in a Hex array, so the Value **"Aastra Telecom  "** (including the two spaces at the end) will become in Hex:
    ```
    0x41,0x61,0x73,0x74,0x72,0x61,0x20,0x54,0x65,0x6c,0x65,0x63,0x6f,0x6d,0x20,0x20
    ```
    ![VCI Win](/posts_images/mitel-windows-vci-02.png)
    ![VCI Win](/posts_images/mitel-windows-vci-03.png)

2) Option 09  
    With this option we give the actual VLAN-ID to the phone, please note that this value needs to be as an **4-Byte Hex** code.
    In this example VLAN-ID **21** will become in Hex: **15** --> translated to a **4-Byte** it will become: **0x0, 0x0, 0x0, 0x15**.
    ![VCI Win](/posts_images/mitel-windows-vci-04.png)

With the options enabled on the scope, it'll look like the following prtscrn:
![VCI Win](/posts_images/mitel-windows-vci-05.png)

**Optionally you can use a VCI Policy on Ms Windows DHCP Server to use this as a wildcard (*) for all AsstraIPPhone types.**
The VCI syntax will become **AastraIPPhone** (without the model type number).

&nbsp;
### VCI (Vendor Class Identifier) Method 2
When using above steps, I did found out that some phone models with the same number, for example Mitel6905 is not working. Probably there is something different in the **firmware they comes by default** or something else is going on like I did notice that some phones actually or **MINET_6905** model, maybe under the hood they do something different because Method 1 was not working for them. Let's take a look at another method that works fine for them.

We can create a **VCI** with following name: **ipphone.mitel.com** Please note the **latest "dot" at the end**, for this you need to end the binary with a null. So add in the left column (Binary Data) at the end **00**.

![VCI minet](/posts_images/mitel_vendor_Class_minet.png)

After that we need to create **option 43** on this VCI.
- Option 43  
    This option will only be applied to our created VCI, so It'll have no impact on other vendors.  
    Chose as type: **String**

    If you only want to pass the correct VLAN-ID to the phone, you can use following string:
    ```
    id:ipphone.mitel.com;vlan=301
    ```
    ![option43 minet](/posts_images/mitel_vendor_option_43_filled_in.png)

With the options enabled on the scope, it'll look like the following prtscrn:
![VCI Win](/posts_images/mitel_vendor_class_dhcp_pool_overview.png)

&nbsp;
### Usefull website to convert VLAN-ID to Hex
When converting the VLAN-ID (in dec to hex) we can use following website: [https://www.rapidtables.com/convert/number/decimal-to-hex.html](https://www.rapidtables.com/convert/number/decimal-to-hex.html)
Please note that Windows will need this as a **4-Byte array**.  
Windows Server is using the **Big endian** format.

Examples:
```
VLAN-ID: 21 --> in Hex this will be: 15
4-Bytes: 0x0, 0x0, 0x0, 0x15
```
![vlan 21](/posts_images/vlan_id_dec_to_hex_01.png)

```
VLAN-ID: 301 --> in Hex this will be: 12D
4-Bytes: 0x0, 0x0, 0x1, 0x2d
```
![vlan 301](/posts_images/vlan_id_dec_to_hex_02.png)

