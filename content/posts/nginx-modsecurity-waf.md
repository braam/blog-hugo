---
title: "Nginx Modsecurity WAF"
date: 2022-02-07T15:21:45+01:00
draft: false
tags:
- Nginx
- WAF
- Linux
---

If you are looking for a good opensource _Web Application Firewall_ (WAF) you'll quickly find modesurity as the ideal candidate. I did succesfuly installed nginx as reverse proxy in combination with modsecurity on my first Raspberry Pi. It only has 1GB RAM but that's enough resources for this project. Below you can find my notes for remembering this installation procedure.

### Install Nginx
First make sure your system is up-to-date:
```bash
sudo apt update && sudo apt upgrade
```

After this we can install nginx:
```bash
sudo apt install nginx
```

&nbsp;
### Compiling Modsecurity 
We'll compile ModSecurity on the rpi itself, this is the libmodsecurity (ModSecurity v3) version. This is the main processor, the modescurity-nginx module (see below) will connect to this main processor.

First get all the necessary tools to be able to compile ModSecurity:
```bash
sudo apt-get install bison build-essential ca-certificates curl dh-autoreconf doxygen \
  flex gawk git iputils-ping libcurl4-gnutls-dev libexpat1-dev libgeoip-dev liblmdb-dev \
  libpcre3-dev libpcre++-dev libssl-dev libtool libxml2 libxml2-dev libyajl-dev locales \
  lua5.3-dev pkg-config wget zlib1g-dev zlibc libxslt-dev libgd-dev
```

Proceed with cloning the repository with git (we'll use the /opt directory for this):
```bash
cd /opt && sudo git clone https://github.com/SpiderLabs/ModSecurity
cd ModSecurity
sudo git submodule init
sudo git submodule update
```

Let's start building ModSecurity (this will take a while on the pi):
```bash
sudo ./build.sh
sudo ./configure
sudo make
sudo make install
```

**Get the ModSecurity-nginx sourcecode from github, to build the dynamic module for nginx:**
```bash
cd /opt && sudo git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```

&nbsp;
### Compiling ModSecurity Nginx Module
Get your installed nginx version number and use it in below commands:
```bash
nginx -v
nginx version: nginx/1.18.0
```
_(We'll continue with version 1.18.0)_

Get the nginx sourcecode so we can start building the dynamic module (use current version):
```bash
cd /opt && sudo wget http://nginx.org/download/nginx-1.18.0.tar.gz
sudo tar -xvzmf nginx-1.18.0.tar.gz
cd nginx-1.18.0
```

It's important to configure the build with the same parameters as how you installed your nginx server. So first check all parameters by running:
```bash
nginx -V
```

Copy all parameters **without the --add-dynamic-module ones**, then add the new dynamic module first followed by the copied parameters for the configure command:

_For Example:_
```bash
sudo ./configure --add-dynamic-module=/opt/ModSecurity-nginx --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-t8A5sG/nginx-1.14.2=. 
-fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-z,relro -Wl,-z,now 
-fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log 
--lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body 
--http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi 
--with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module 
--with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module 
--with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module 
--with-mail=dynamic --with-mail_ssl_module
```

_**Note:**_ If your nginx server is installed without any extra dynamic modules already, you can execute following command, so you do not have to copy all parameters like above:
```bash
sudo ./configure --add-dynamic-module=/opt/ModSecurity-nginx `nginx -V`
```

After this let's make the module:
```bash
sudo make modules
```

After the build copy the fresh module into nginx share folder:
```bash
sudo cp objs/ngx_http_modsecurity_module.so /usr/share/nginx/modules
```

&nbsp;
### Install CRS rules (OWASP)
To be able to detect injection strings or other attacks, modsecurity uses rules for this. Lucky for us OWASP has some corerules set we can use for free. First install modsecurity-crs default rules:
```bash
sudo apt install modsecurity-crs
```

Remove the default rules and replace them with the OWASP corerules (in /usr/local):
```bash
sudo rm -rf /usr/share/modsecurity-crs
sudo git clone https://github.com/coreruleset/coreruleset /usr/local/modsecurity-crs
#Use the example configs by renaming them.
sudo mv /usr/local/modsecurity-crs/crs-setup.conf.example /usr/local/modsecurity-crs/crs-setup.conf
sudo mv /usr/local/modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example /usr/local/modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
```

<<TODO: adding automatic OWASP rules updates, probably by using sudo git pull inside the directory. >>

&nbsp;
### Nginx Config Changes
Let nginx know we'll want to use the ModSecurity dynamic module:
```
change /etc/nginx/nginx.conf
add before events block: load_module /usr/share/nginx/modules/ngx_http_modsecurity_module.so;
```
![nginx.conf](/posts_images/nginx-modsecurity-01.png)

Create a directory to store our modsecurity files:
```bash
sudo mkdir /etc/nginx/modsec
sudo cp /opt/ModSecurity/unicode.mapping /etc/nginx/modsec
sudo cp /opt/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
```

With a text editor such as vim, open **/etc/nginx/modsec/modsecurity.conf** and change the value for **SecRuleEngine** to **On**
![nginx.conf](/posts_images/nginx-modsecurity-02.png)

Create the main conf file:
```bash
sudo vi /etc/nginx/modsec/main.conf
#Add following lines:
Include /etc/nginx/modsec/modsecurity.conf
Include /usr/local/modsecurity-crs/crs-setup.conf
Include /usr/local/modsecurity-crs/rules/*.conf
```

The last step is to enable modsecurity for your website, you can choose for which website you want this. 
```bash
sudo vi /etc/nginx/sites-available/default
#Add in each server block following lines if you want to enable modsecurity for them.
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/main.conf;
```

Restart nginx server to initialse the config changes:
```bash
sudo systemctl restart nginx.service
```

&nbsp;
### Test ModSecurity
Test ModSecurity by performing a simple local file inclusion attack by running the following command:
```bash
curl http://<SERVER-IP/DOMAIN>/index.html?exec=/bin/bash
```
If ModSecurity has been configured correctly and is actively blocking attacks, the following error is returned:
```html
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>
```

We can see the violation audit logs (default behaviour):
```
sudo cat /var/log/modsec_audit.log
```

&nbsp;
### Disable Specific Rule on Website
Sometimes ModSecurity is not correc tin determining an attack, therefore it will block some requests that we do not want. We can disable a specific matched rule (you can find it in the /var/log/modsec_audit.log file) for a location:
```bash
server {
    server_name test.example.com;
    modsecurity on;
    . . .

    location /api.php {
        modsecurity_rules '
            SecRuleRemoveById 941160
        ';
    }
}
```
