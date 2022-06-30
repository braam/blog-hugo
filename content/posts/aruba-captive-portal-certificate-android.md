---
title: "Aruba Captive Portal Certificate Android"
date: 2022-06-30T15:26:40+02:00
draft: false
tags:
- Aruba
- Wifi
---

When implementing a captive portal with a certificate (and the dynamic FQDN) we do notice a certificate warning on android devices, iOS or desktop browser are not throwing any error at all.
I tried a few settings and config changes but nothing is solving are problem.

Time to use google! I found following interested article: [https://developer.android.com/training/articles/security-ssl](https://developer.android.com/training/articles/security-ssl) What is applying to our problem:

### Missing intermediate certificate authority
The third case of SSLHandshakeException occurs due to a missing intermediate CA. Most public CAs don't sign server certificates directly. Instead, they use their main CA certificate, referred to as the root CA, to sign intermediate CAs. They do this so the root CA can be stored offline to reduce risk of compromise. However, operating systems like Android typically trust only root CAs directly, which leaves a short gap of trust between the server certificate—signed by the intermediate CA—and the certificate verifier, which knows the root CA. To solve this, the server doesn't send the client only it's certificate during the SSL handshake, but a chain of certificates from the server CA through any intermediates necessary to reach a trusted root CA.

To see what this looks like in practice, here's the mail.google.com certificate chain as viewed by the **openssl** s_client command:

```bash
$ openssl s_client -connect mail.google.com:443
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google LLC/CN=mail.google.com
   i:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA
 1 s:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA
   i:/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
---
```

This shows that the server sends a certificate for mail.google.com issued by the Thawte SGC CA, which is an intermediate CA, and a second certificate for the Thawte SGC CA issued by a Verisign CA, which is the primary CA that's trusted by Android.

However, it is not uncommon to configure a server to not include the necessary intermediate CA. For example, here is a server that can cause an error in Android browsers and exceptions in Android apps:

```bash
$ openssl s_client -connect egov.uscis.gov:443
---
Certificate chain
 0 s:/C=US/ST=District Of Columbia/L=Washington/O=U.S. Department of Homeland Security/OU=United States Citizenship and Immigration Services/OU=Terms of use at www.verisign.com/rpa (c)05/CN=egov.uscis.gov
   i:/C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=Terms of use at https://www.verisign.com/rpa (c)10/CN=VeriSign Class 3 International Server CA - G3
---
```

What is interesting to note here is that visiting this server in most desktop browsers does not cause an error like a completely unknown CA or self-signed server certificate would cause. This is because most desktop browsers cache trusted intermediate CAs over time. Once a browser has visited and learned about an intermediate CA from one site, it won't need to have the intermediate CA included in the certificate chain the next time.

Some sites do this intentionally for secondary web servers used to serve resources. For example, they might have their main HTML page served by a server with a full certificate chain, but have servers for resources such as images, CSS, or JavaScript not include the CA, presumably to save bandwidth. Unfortunately, sometimes these servers might be providing a web service you are trying to call from your Android app, which is not as forgiving.

To fix this issue, configure the server to include the intermediate CA in the server chain. Most CAs provide documentation on how to do this for all common web servers.


### Create the correct Certificate
So with the above explanation we need a certificate in the following form:
![cert form](/posts_images/aruba_cert_structure.png)

We do need the root bundle for this, in our example we are using a certificate issued by GoDaddy, we can find the whole certificate bundle here: [https://certs.godaddy.com/repository](https://certs.godaddy.com/repository)
(you'll need the full one, gd_bundle-g2-g1.crt)

**Convert a PFX to PEM**
Using openssl we can achieve this:
```bash
$ openssl pkcs12 -in cert2022.pfx -out public.pem -nokeys
$ openssl pkcs12 -in cert2022.pfx -out file.withkey.pem
$ openssl rsa -in file.withkey.pem -out private.key
#Combine the certificate
$ cat public.pem gd_bundle-g2-g1.crt private.key > fullchain.pem
```


### Configure Aruba VirtualController
You'll first need a SSID for the captive portal, we'll use in this example an Open SSID testBKM set Primary Usage as **Guest**.
![SSID testBKM](/posts_images/aruba_new_network_1.png)
![SSID testBKM](/posts_images/aruba_new_network_2.png)

Proceed by uploading the certificate:
![cert management](/posts_images/aruba_certificate_management.png)
![cert mangement](/posts_images/aruba_upload_server_certificate.png)

Important is to first upload the certificate as a Server certificate and in the certificate assignment use that certificate for the captive portal.
*(below picture is containing the name full_chain_bkm.be, but should be in this example fullchain_bkm.be)*
![cert mangement](/posts_images/aruba_certificate_assignment.png)
