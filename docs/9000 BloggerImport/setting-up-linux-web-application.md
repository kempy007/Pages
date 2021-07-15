---
title: 'Setting up a Linux Web Application Firewall Using Applicure DotDefender'
date: 2014-08-29T14:39:00.000+01:00
draft: false
aliases: [ "/2014/08/setting-up-linux-web-application.html" ]
parent: Blogger
---
#### date: 2014-08-29

WAF Node Build notes

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  

Install minimum ubuntu(13.04) with openssh selected.

Set IP 192.168.10.220/24

GW 192.168.10.1

DNS 208.67.222.222  & 208.67.220.220

  

apt-get update

apt-get upgrade

apt-get install apache2 sqlite3 libcgi-pm-perl

a2enmod proxy proxy\_http ssl

  

  

mkdir /etc/apache2/ssl

cd /etc/apache2/ssl

\# generate signing request

openssl genrsa -des3 -out server.key 2048

openssl req -new -key server.key -out server.csr

  

\# self sign request

openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

  

#to remove password from private key so service restart does not require assistance

openssl rsa -in server.key -out server.key.nopass

  

#copy /etc/apache2/sites-available to \*-bak

  

#symlink in the ssl site config

ln -s /etc/apache2/sites-available/default-ssl /etc/apache2/sites-enabled/000-default-ssl

  

nano /etc/apache2/sites-available/default

#-----------------------------------------------------------

#---------- Working HTTP Vhost for redirect logic ----------

#-----------------------------------------------------------

<VirtualHost \*:80>

        UseCanonicalName off

        ProxyPreserveHost on

        ProxyRequests off

        <Proxy \*>

                Order deny,allow

                Allow from all

        </Proxy>

        RewriteEngine on

#must be prior to www check and must go to secure else loop occurs

        RewriteCond %{HTTP\_HOST} ^secure

        RewriteRule (.\*) https://%{HTTP\_HOST}%{REQUEST\_URI} \[R=301,L\]

  

        RewriteCond %{HTTP\_HOST} ^checkout

        RewriteRule (.\*) https://%{HTTP\_HOST}%{REQUEST\_URI} \[R=301,L\]

  

  

  

        RewriteCond %{HTTP\_HOST} !^www

        RewriteRule (.\*) http://www\\.%{HTTP\_HOST}%{REQUEST\_URI} \[R=301,L\]

  

  

        RewriteCond %{HTTPS} off \[NC\]

        RewriteCond %{REQUEST\_URI} ^/secure/(.\*) \[NC\]

        RewriteRule ^(.\*)$ https://%{HTTP\_HOST}%{REQUEST\_URI} \[R=301,L\]

  

  

#       RewriteCond %{HTTPS} off

#       RewriteRule (.\*) https://%{HTTP\_HOST}%{REQUEST\_URI} \[R=301,L\]

        ProxyPass / http://192.168.10.15/

        ProxyPassReverse / http://192.168.10.15/

</VirtualHost>

  

<VirtualHost \*:80>

        ServerName error.somedomain.co.uk

        UseCanonicalName off

        ProxyPreserveHost on

        ProxyRequests off

        <Proxy \*>

                Order deny,allow

                Allow from all

        </Proxy>

        ProxyPass / http://192.168.10.15/

        ProxyPassReverse / http://192.168.10.15/

</VirtualHost>

  

<VirtualHost \*:80>

        ServerName extranet. somedomain.co.uk

        ServerAlias bi. somedomain.co.uk

        UseCanonicalName off

        ProxyPreserveHost on

        ProxyRequests off

        <Proxy \*>

                Order deny,allow

                Allow from all

        </Proxy>

        RewriteEngine On

        RewriteCond %{HTTPS} off

        RewriteRule (.\*) https://%{HTTP\_HOST}%{REQUEST\_URI} \[R=301,L\]

        ProxyPass / http://172.16.1.16/

        ProxyPassReverse / http://172.16.1.16/

</VirtualHost>

  

  

<VirtualHost \*:80>

        ServerName WAFCN1

        ServerAdmin webmaster@localhost#

        DocumentRoot /var/www

        <Directory />

                Options FollowSymLinks

                AllowOverride None

        </Directory>

        <Directory /var/www/>

                Options Indexes FollowSymLinks MultiViews

                AllowOverride None

                Order allow,deny

                allow from all

        </Directory>

  

        ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/

        <Directory "/usr/lib/cgi-bin">

                AllowOverride None

                Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch

                Order allow,deny

                Allow from all

        </Directory>

  

        ErrorLog ${APACHE\_LOG\_DIR}/error.log

  

        # Possible values include: debug, info, notice, warn, error, crit,

        # alert, emerg.

        LogLevel warn

  

        CustomLog ${APACHE\_LOG\_DIR}/access.log combined

</VirtualHost>

  

#-------------------------------------------------------

#----- End of file -------------------------------------

#-------------------------------------------------------

  

  

nano /etc/apache2/sites-available/default-ssl

#-----------------------------------------------------------

#---------- Working HTTPs Vhosts needs SNI handling  -------

#-----------------------------------------------------------

#NameVirtualHost \*:443

<IfModule mod\_proxy.c>

<VirtualHost \*:443>

\# Catch anyhost here and send to error vip on load balancer

#SSLStrictSNIVHostCheck on

        UseCanonicalName off

        SSLEngine On

        SSLProxyEngine on

        ProxyPreserveHost On

  

        SSLProtocol -ALL +SSLv3 +TLSv1

        SSLHonorCipherOrder On

        SSLCipherSuite RC4-SHA:HIGH:!ADH

  

        SSLCertificateFile /etc/apache2/ssl/server.crt

        SSLCertificateKeyFile /etc/apache2/ssl/server.key.nopass

        ProxyPass / http://192.168.10.7/

        ProxyPassReverse / http://192.168.10.7/

</VirtualHost>

  

  

<VirtualHost \*:443>

\# just a test site

        ServerName www.host. somedomain.co.uk

        SSLEngine On

        SSLProxyEngine on

        ProxyPreserveHost On

  

        SSLProtocol -ALL +SSLv3 +TLSv1

        SSLHonorCipherOrder On

        SSLCipherSuite RC4-SHA:HIGH:!ADH

  

        SSLCertificateFile /etc/apache2/ssl/www.host.test.crt

        SSLCertificateKeyFile /etc/apache2/ssl/www.host.test.key.nopass

        ProxyPass / http://192.168.10.15/

        ProxyPassReverse / http://192.168.10.15/

</VirtualHost>

  

  

</IfModule>

  

#-------------------------------------------------------

#----- End of file -------------------------------------

#-------------------------------------------------------

  

nano /etc/apache2/ports.conf

#------------------------------------------

#------- start of file     ----------------

#------------------------------------------

NameVirtualHost \*:80

Listen 80

  

<IfModule mod\_proxy.c>

    NameVirtualHost \*:443

    Listen 443

</IfModule>

  

<IfModule mod\_ssl.c>

</IfModule>

  

<IfModule mod\_gnutls.c>

    Listen 443

</IfModule>

#-------------------------------------------------------

#----- End of file -------------------------------------

#-------------------------------------------------------

  

nano /etc/apache2/mod-available/proxy.conf

#------------------------------------------

#------- start of file     ----------------

#------------------------------------------

<IfModule mod\_proxy.c>

  

ProxyRequests Off

<Proxy \*>

        AddDefaultCharset off

        Order deny,allow

        Allow from all

        #Allow from .example.com

</Proxy>

  

ProxyVia On

  

</IfModule>

#-------------------------------------------------------

#----- End of file -------------------------------------

#-------------------------------------------------------

  

  

  

Install Applicure DotNetDefender (install documentation available from website also)

  

make sure to remove any previous install # rm -r /usr/local/APPCure-full/

  

copy over the dotDefender-5.10.Linux.x86\_64.deb.bin via winSCP and 

\# chmod 777 dotDefender-5.10.Linux.x86\_64.deb.bin

./dotDefender-5.10.Linux.x86\_64.deb.bin

@Next

@I agree

@next

enter; /usr/sbin/apache2  @next

@next

enter; dotDefender @next

enter the pw @next

select Auto @next

select 1 day @next

select Applicure @next

@next

@go

  

DotDefender Administration, Ensure your hosts files has entries for the nodes. Then browse to http://WAFCN\[1 or 2\]/dotDefender

Login as admin, using the common pw. You will have to start apache on the backup node to backup and restore setting between nodes. This is something that can be improved! Tried to do it in SVN but that breaks the backup node and dotdefender has to be reinstalled. So for now you have to backup and restore via the web gui.