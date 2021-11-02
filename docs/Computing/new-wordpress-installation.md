# New Wordpress installation

## Create a new MySQL database

``` sql
CREATE DATABASE wp_lindyhoppeln CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'lindyhoppeln'@'localhost';
GRANT ALL PRIVILEGES ON wp_lindyhoppeln.* To 'lindyhoppeln'@'localhost' IDENTIFIED BY 'passphrase';
```

utf8mb4 is the “real” Unicode, MySQL is using an old, buggy own variant.

The database name must not contain dashes (or those must be quoted).

## Wordpress

For Wordpress the following is my default choice:

* Theme: Twenty Sixteen
* Plugins:
	* Autoptimize: compress, use system fonts instead of Google fonts
	* Prosodia VGW OS: counting pixels for VG Wort (only relevant to German-language weblogs)
	* Site Kit by Google: sitemap
	* WP Super Cache: generating static web pages for caching
	* Yoast SEO: search-engine optimization (e.g. meta tags)
	* Antispam Bee: free anti-spam plugin

## Configure a new domain

First nginx, in sites-available:

``` nginx
server {
        listen 80;
      	listen [::]:80;
        server_name www.thomas-huehn.de;
        location /.well-known/acme-challenge {
		root /var/www/thomas-huehn/html;
	}
}

server {
        listen 80;
      	listen [::]:80;
        server_name thomas-huehn.de;
        root /var/www/thomas-huehn/html;
        index index.php index.html;
        location /.well-known/acme-challenge {
      	        root /var/www/thomas-huehn/html;
        }
}
```

Then acme.sh:

``` bash
acme.sh --issue --nginx -d www.thomas-huehn.de -d thomas-huehn.de
acme.sh --install-cert -d www.thomas-huehn.de --key-file /etc/cert-files/key-www.thomas-huehn.de.pem --fullchain-file /etc/cert-files/cert-www.thomas-huehn.de.pem --reloadcmd "service nginx force-reload"
```
 Check whether nginx has access to /etc/cert-files/ and reload:
 
 ```bash
 systemctl reload nginx
 ```
 
 Then fill in actual root sextion, in case of Wordpress:
 
``` nginx
root /var/www/thomas-huehn/html;
index index.php index.html;
location / {
      try_files $uri $uri/ /index.php?$args;
}
location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      if ($uri !~ "^/uploads/") {
	      fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
      }
}
location ~ ^/wp-json/ {
      # if permalinks not enabled
      rewrite ^/wp-json/(.*?)$ /?rest_route=/$1 last;
}
```
 
 And a redirect:
 
``` nginx
 location / {
         return 301 https://www.thomas-huehn.de$request_uri;
}
```
 
 Then correct nginx configuration with redirect no-www to www and TLS:
 
``` nginx
 server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name www.thomas-huehn.de;
	ssl_certificate         /etc/cert-files/cert-www.thomas-huehn.de.pem;
	ssl_certificate_key     /etc/cert-files/key-www.thomas-huehn.de.pem;
	ssl_protocols           TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDH
	root /var/www/thomas-huehn/html;
	index index.php index.html;
	location / {
		try_files $uri $uri/ /index.php?$args;
	}
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
		if ($uri !~ "^/uploads/") {
			fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
		}
	}
	location ~ ^/wp-json/ {
		# if permalinks not enabled
		rewrite ^/wp-json/(.*?)$ /?rest_route=/$1 last;
	}
}
server {
	listen 80;
	listen [::]:80;
	server_name www.thomas-huehn.de;
	location /.well-known/acme-challenge {
		root /var/www/thomas-huehn/html;
	}
	location / {
		return 301 https://www.thomas-huehn.de$request_uri;
	}
}
server {
	listen 80;
	listen [::]:80;
	server_name thomas-huehn.de;
	location /.well-known/acme-challenge {
		root /var/www/thomas-huehn/html;
	}
	location / {
		return 301 https://www.thomas-huehn.de$request_uri;
	}
}
```
 
 Calling acme.sh with --test targets Letsencrypt's staging server.
 
## Real cron for Wordpress
 
``` bash
crontab -u www-data -e
```

```
* * * * * /usr/bin/php  /var/www/thomas-huehn/html/wp-cron.php
* * * * * /usr/bin/php  /var/www/lindyhoppeln/html/wp-cron.php
```
 
 In wp-config-php. pretty much at the top:
 
``` php
/** Disable virtual cron */
define('DISABLE_WP_CRON', true);
```
 
## Repair pingback
 
In wp-includes/cron.php: timeout 1 instead of 0.01:
 
``` php
$cron_request = apply_filters(
             'cron_request',
             array(
                     'url'  => add_query_arg( 'doing_wp_cron', $doing_wp_cron, site_url( 'wp-cron.php' ) ),
                     'key'  => $doing_wp_cron,
                     'args' => array(
                             'timeout'   => 1,
```
 