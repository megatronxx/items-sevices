# items-sevices

## Enable SSL in Apache (OSX)

The following will guide you through the process of enabling SSL on a Apache webserver
- The instructions have been verified with OSX El Capitan (10.11.2) running **Apache 2.4.16**
- The instructions assume you already have a basic Apache configuration enabled on OSX, if this is not the case feel free to consult Gist: "[Enable Apache HTTP server (OSX)](http://)"

#### Apache SSL Configuration

Create a directory within `/etc/apache2/` using **Terminal**.app: `sudo mkdir /etc/apache2/ssl`  
Next, generate two host keys:
```
sudo openssl genrsa -out /etc/apache2/server.key 2048
sudo openssl genrsa -out /etc/apache2/ssl/localhost.key 2048
sudo openssl rsa -in /etc/apache2/ssl/localhost.key -out /etc/apache2/ssl/localhost.key.rsa
```

Create a configuration file using **Terminal**.app: `sudo touch /etc/apache2/ssl/localhost.conf`  
Edit the newly created configuration file and add the following:
```
[req]
default_bits = 1024
distinguished_name = req_distinguished_name
req_extensions = v3_req

[req_distinguished_name]

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = *.localhost
```

Generate the required Certificate Requests using **Terminal**.app:
```
sudo openssl req -new -key /etc/apache2/server.key -subj "/C=/ST=/L=/O=/CN=/emailAddress=/" -out /etc/apache2/server.csr
sudo openssl req -new -key /etc/apache2/ssl/localhost.key.rsa -subj "/C=/ST=/L=/O=/CN=localhost/" -out /etc/apache2/ssl/localhost.csr -config /etc/apache2/ssl/localhost.conf
```
**Note**: Complete the values `C= ST= L= O= CN=` to reflect your own organizational structure, where:
* `C=` eq. Country: The two-letter ISO abbreviation for your country.
* `ST=` eq. State or Province: The state or province where your organization is legally located.
* `L=` eq. City or Locality: The city where your organization is legally located.
* `O=` eq. Organization: he exact legal name of your organization.
* `CN=` eq. Common Name: The fully qualified domain name for your web server


Use the Certificate Requests to sign the SSL Certificates using **Terminal**.app:
```
sudo openssl x509 -req -days 365 -in /etc/apache2/server.csr -signkey /etc/apache2/server.key -out /etc/apache2/server.crt
sudo openssl x509 -req -extensions v3_req -days 365 -in /etc/apache2/ssl/localhost.csr -signkey /etc/apache2/ssl/localhost.key.rsa -out /etc/apache2/ssl/localhost.crt -extfile /etc/apache2/ssl/localhost.conf
```

Add the SSL Certificate to **Keychain Access**.
```
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /etc/apache2/ssl/localhost.crt
```

#### Apache Configuration
Edit the Apache main configuration file `/etc/apache2/httpd.conf` and enable the required modules to support SSL :
```
LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
LoadModule ssl_module libexec/apache2/mod_ssl.so
```

Enable Secure (SSL/TLS) connections
```
Include /private/etc/apache2/extra/httpd-ssl.conf
```

#### Apache Virtual Host Configuration
Edit the Virtual Hosts file `/etc/apache2/extra/httpd-vhosts.conf` and add the SSL Directive at the end of the file:
```
<VirtualHost *:443>
    ServerName localhost
    DocumentRoot "/Library/WebServer/Documents"

    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /etc/apache2/ssl/localhost.crt
    SSLCertificateKeyFile /etc/apache2/ssl/localhost.key

    <Directory "/Library/WebServer/Documents">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```

Finally restart Apache using **Terminal**.app : `sudo apachectl restart`  
Open Safari and visit [https://localhost](https://localhost) to verify your configuration.
