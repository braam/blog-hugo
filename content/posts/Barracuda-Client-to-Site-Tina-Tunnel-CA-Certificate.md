---
title: "Barracuda Client to Site Tina Tunnel CA Certificate"
date: 2022-07-15T08:22:53+02:00
draft: false
tags:
- Barracuda
---

When using the Barracuda Client-to-Site tunnel, we have a few options for the certificate. 
1) We can use a self-signed certificate on the firewall (by default warnings on client)
2) We can use signed certficate from a CA on the firewall (no warnings on client)

Following campus article can be used as a reference: [https://campus.barracuda.com/product/cloudgenfirewall/doc/79462913/how-to-set-up-external-ca-vpn-certificates/](https://campus.barracuda.com/product/cloudgenfirewall/doc/79462913/how-to-set-up-external-ca-vpn-certificates/)

### Install root Certificate
Sometimes the Barracuda NGF does not have the root CA certificate installed by default, if this is the case you can upload the root certificate for that CA on the firewall:
1) Go to **CONFIGURATION** > **Configuration Tree** > **Box** > **Assigned Services** > **VPN-Service** > **VPN Settings**.
2) In the left menu, select **Root Certificates**.
3) Click **Lock**.
4) **Right-click** the table and select **Import PEM from File** or **Import CER from File**.
5) Select the file containing the root certificate and click **Open**. The **Root Certificate** window opens.
6) Verify that the window displays the tab **Certificate details**.
7) Enter a **Name**. This is the name that is displayed for this certificate throughout the VPN configuration.
8) Select the **Usage**.
   - **Barracuda Personal** – Select to use this certificate for client-to-site VPN using the TINA protocol.

![Root-CA-Certificate](/posts_images/tina-client-to-site-ca_03.png)

&nbsp;
### Install the Server Certificate
We'll configure the VPN Settings with a **signed certificate by the CA**. It's important to note, that we do need this in **PEM format**.
Following is required:
- Private key
  - private key in RSA format.
- Public Certificate
  - public certificate, can be a wildcard also.
- Full Chain
  - Contains in order: public certificate + root certificate + intermediate certificate + private key.

**OPTIONAL - Convert a PFX to PEM**  
    We do need the root bundle for this, in our example we are using a certificate issued by GoDaddy, we can find the whole certificate bundle here: [https://certs.godaddy.com/repository](https://certs.godaddy.com/repository)
    (you'll need the full one, gd_bundle-g2-g1.crt)

    Using openssl we can achieve this:
    ```bash
    $ openssl pkcs12 -in cert2022.pfx -out public.pem -nokeys
    $ openssl pkcs12 -in cert2022.pfx -out file.withkey.pem
    $ openssl rsa -in file.withkey.pem -out private.key
    #Combine the certificate
    $ cat public.pem gd_bundle-g2-g1.crt private.key > fullchain.pem
    ```

To upload the certificate, proceed in the Firewall Admin:
1) Go to **CONFIGURATION** > **Configuration Tree** > **Box** > **Assigned Services** > **VPN-Service** > **VPN Settings**.
2) In the left menu, select **General**.
3) Click **Lock**.
4) Import the Private key.
5) Import the Certificate.
6) Imoprt the fullchain.
7) Click **Send Changes** and **Activate**.

![Tina-CA-Certificate](/posts_images/tina-client-to-site-ca_01.png)

**Please note**: When you do not see the **Chain option**, upgrade your Firewall Admin as your version is not able to upload this. When you cannot upload the fullchain, your NAC client will still display an invalid certificate warning.

&nbsp;
### Create a Service key
1) Go to **CONFIGURATION** > **Configuration Tree** > **Box** > **Assigned Services** > **VPN-Service** > **VPN Settings**.
2) In the left menu, select **Service Keys**.
3) Click **Lock**.
4) Right-click the table and select **New Key**. 
5) Enter a **Key Name** and click **OK**.
6) Select the **Key Length** and click **OK**.
7) Click **Send Changes** and **Activate**.
