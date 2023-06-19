---
title: "Barracuda Recover TOTP User"
date: 2023-02-20T15:20:14+01:00
draft: false
tags:
- Barracuda
---

When a user does self-enrollment he needs to create a TOTP link by scanning the corresponding QR code with his authentication app. After that the Barracuda firewall will remember this user together with the possible backup/recovery codes.
But what can we do when a user lost his phone or the authenticator app is not working like it should?

### Retrieve a backup/recovery TOTP code
Remember, when first initiating the self-enrollment with the authenticator app, the Barracuda firewall did create some backup/recovery codes. Luckily for use we can retrieve those codes
when the users did not backup it themselves. Proceed by logging in on the Barracuda firewall through SSH:

```bash
$ cd /opt/phion/config/active
$ cat authenticationschemes_dyndata.conf
```
	
Something like the following output will appear:

```
MFA_EMAIL_BODY_TEMPLATE = -----BEGIN Text-----
<html>
<h1>TOTP Enrollment Automatic E-mail</h1>
<p>You have been setup by your system administrator to use an extra Time-based One-time-password (OTP) when logging into the Barracuda Network Access (VPN) Client, CudaLaunch or Web Portal. These TOTPs can be setup and generated using the TOTP App of your choice (e.g. Google Authenticator, Microsoft Authenticator or any other App that supports the protocol).</p>
<p>Please setup TOTP on your device (e.g. mobile phone) either by manually entering the secret key shown below or by scanning the QR code.</p>
<p>Backup codes are also provided below. Each Backup Code can be used once to access your account instead of a TOTP (for example if you lose your device).</p>
<p>Secret Key: {{QRCODEPLAINTEXT}}</p>
<p>{{QRCODEIMAGE}}</p>
<p>Once you have set this up, you can use your App to generate the TOTP you will be prompted for when logging into the Barracuda Network Access (VPN) Client, CudaLaunch or Web Portal.</p>
<p>Backup codes: {{SCRATCHCODES}}</p>
<p>Best Regards</p>
</html>
-----END Text-----


[googleauthdata_test.user]
NAME = test.user
SCRATCHCODES = 12345678|87654321|xxxxx|xxxxx
SECRETKEY = ABCDEFGHIJKLMNOPQRSTUVWXYZ
USEDTOTPS = 55841731=990405|55841736=500429|55841762=338147


.
```

As you can see the **SCRATCHCODES** are the backup/recovery codes created during linking the TOTP authenticator app.
You can use every code once, after logging in with a backup/recovery code you can do the enrollment process again.

These backup/recovery codes are also searchable inside the backup PAR file.
