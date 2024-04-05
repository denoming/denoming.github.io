---
title: "Install Apache Web Server with HTTPS support"
date: 2020-09-05
categories: [Engineering,Linux]
tags: [apache,https,web-server]
---

Initial:
* OS version – CentOS Linux release 7.4.1708 (Core)
* Core version – Linux 3.10.0-693.17.1.el7.x86_64

## Step 1 – Install prerequests

```bash
$ sudo yum install epel-release
$ sudo yum install httpd mod_ssl python-certbot-apache
```

## Step 2 – Configure Apache server

Run httpd daemon:
```bash
$ sudo systemctl start httpd
$ sudo systemctl status httpd
httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2018-04-28 19:10:47 EEST; 1h 1min ago
     Docs: man:httpd(8)
```

Configure HTTP/HTTPS access on firewall:
```bash
$ sudo firewall-cmd --add-service=http
$ sudo firewall-cmd --add-service=https
$ sudo firewall-cmd --runtime-to-permanent
```

Check access thought 80 (HTTP) and 443 (HTTPS) ports:
```bash
$ curl example.com
...
$ curl -k https://example.com
...
```
You should see HTML content of main page in both cases.

Example of main configuration file _/etc/httpd/conf/httpd.conf_:
```
# Configure server root
ServerRoot "/etc/httpd"
 
# Configure port to listen
Listen 80
 
# Dynamic Shared Object (DSO) Support
Include conf.modules.d/*.conf
 
# Configure MPM
<IfModule prefork.c>
    StartServers 4
    MinSpareServers 3
    MaxSpareServers 10
    ServerLimit 256
    MaxClients 256
    MaxRequestsPerChild 10000
</IfModule>
 
# Change apache process user/group
User apache
Group apache
 
# Redirect to main virtual host
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / http://www.example.com/
</VirtualHost>
 
# Deny access to the entirety of server's filesystem
<Directory />
    AllowOverride none
    Require all denied
</Directory>
 
# Configure default directory index
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
 
# Prevent to read .htaccess file
<Files ".ht*">
    Require all denied
</Files>
 
# Configre log
ErrorLog "logs/error_log"
LogLevel warn
<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog "logs/access_log" combined
</IfModule>
 
# Configure MIME types
<IfModule mime_module>
    TypesConfig /etc/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>
 
<IfModule mime_magic_module>
    MIMEMagicFile conf/magic
</IfModule>
 
# Specify a default charset for all content served
AddDefaultCharset UTF-8
 
# EnableMMAP and EnableSendfile
EnableSendfile on
 
IncludeOptional conf.d/*.conf
```

Example of ssl configuration file _/etc/httpd/conf.d/ssl.conf_:
```
Listen 443 https

# Type of pass phrase dialog for encrypted private keys
SSLPassPhraseDialog exec:/usr/libexec/httpd-ssl-pass-dialog

# Configure SSL Session Cache
SSLSessionCache         shmcb:/run/httpd/sslcache(512000)
SSLSessionCacheTimeout  300

# Pseudo Random Number Generator (PRNG) seeding source
SSLRandomSeed startup file:/dev/urandom  256
SSLRandomSeed connect builtin

# Cryptographic hardware accelerator
SSLCryptoDevice builtin

# Configure available cipher suite for negotiation in SSL handshake
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH

# Configure available SSL protocols
SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLHonorCipherOrder On

# Enable header modification
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff

# Disable SSL compression
SSLCompression off

# Enable stapling of OCSP responses in the TLS handshake
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
```

Example of virtual host configuration in file _/etc/httpd/conf.d/www.example.conf_:
```
<VirtualHost *:80>
    ServerName www.example.com
    ServerAdmin admin@example.com
    DocumentRoot /var/www/example.com/www
    <Directory "/var/www/example.com/www">
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```
As you can see files of virtual were placed in _/var/www/example.com/www_ dir. Our configuration isn’t secure yet.

## Step 3 – Request SSL certificate from Lets Encrypt

For multi domain system (example.com – base domain, another are sub-domains):
```bash
$ sudo certbot --apache -d example.com -d www.example.com -d blog.example.com
```
To execute the interactive installation with prompt information about domain:
```bash
$ sudo certbot --apache
```
In this mode you should provide domain information, email address for lost key recovery and another information.
The generated certificate files should be available within a subdirectory named after your base domain in the /etc/letsencrypt/live directory.

## Step 4 – Configure secure virtual host

Edit main configuration file _/etc/httpd/conf/httpd.conf_:
```bash
...
<VirtualHost *:80>
    ServerName denoming.in.ua
    Redirect permanent / https://www.denoming.in.ua/ # change http to https
</VirtualHost>
...
```
Edit virtual host configuration file _/etc/httpd/conf.d/www.example.conf_:
```
<VirtualHost *:80>
    ServerName www.example.com
    Redirect permanent / https://www.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.example.com
    ServerAdmin admin@example.com
    DocumentRoot /var/www/example.com/www
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
    <Directory "/var/www/example.com/www">
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```
We have specified SSL options and append HTTP redirection. Now our virtual host is secure.

## Step 5 – Check certificate status

You can verify the status if your SSL certificate with thw following link:
```
https://www.ssllabs.com/ssltest/analyze.html?d=example.com
```

## Step 6 – Configure auto renew of certificate

Append cron job for certificate renew:
```bash
$ crontab -e
30 2 * * * /usr/bin/certbot renew >> /var/log/certbot.log
```
