# Server-Notes
## NGINX Proxy Manager

*Configuration*

 1. Unraid Template ![Unraid Settings](https://github.com/ShyViolets/Server-Notes/blob/main/nginx_template_settings.png?raw=true)
 2. Configuration File Edits
In the WebUI, paste the following into the “Advanced” tab for the nextcloud proxy.

         location /.well-known/carddav {    
        return 301 $scheme://$host/remote.php/dav;}
        location /.well-known/caldav {    
        return 301 $scheme://$host/remote.php/dav;}
        client_max_body_size 16G;
        client_body_buffer_size 500M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;
In Full config under proxyhost...
```
server {
  set $forward_scheme http;
  set $server         "192.168.1.112";
  set $port           444;

  listen 80;
listen [::]:80;

listen 443 ssl http2;
listen [::]:443 ssl http2;


  server_name your.server.com;


  # Custom SSL
  ssl_certificate /data/custom_ssl/npm-1/fullchain.pem;
  ssl_certificate_key /data/custom_ssl/npm-1/privkey.pem;




# Asset Caching
  include conf.d/include/assets.conf;


  # Block Exploits
  include conf.d/include/block-exploits.conf;



  # HSTS (ngx_http_headers_module is required) (63072000 seconds = 2 years)
  add_header Strict-Transport-Security "max-age=63072000;includeSubDomains; preload" always;





    # Force SSL
    include conf.d/include/force-ssl.conf;




proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection $http_connection;
proxy_http_version 1.1;


  access_log /data/logs/proxy-host-3_access.log proxy;
  error_log /data/logs/proxy-host-3_error.log warn;

location /.well-known/carddav {    
    return 301 $scheme://$host/remote.php/dav;}

location /.well-known/caldav {    
    return 301 $scheme://$host/remote.php/dav;}


client_max_body_size 16G;
client_body_buffer_size 1024M;
client_body_timeout 3000s;
fastcgi_buffers 64 4K;
proxy_read_timeout 310s;
fastcgi_request_buffering off;





  location / {





  # HSTS (ngx_http_headers_module is required) (63072000 seconds = 2 years)
  add_header Strict-Transport-Security "max-age=63072000;includeSubDomains; preload" always;





    
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    proxy_http_version 1.1;
    

    # Proxy!
    include conf.d/include/proxy.conf;
  }


  # Custom
  include /data/nginx/custom/server_proxy[.]conf;
}
```
## Nextcloud Configuration
  * All paths should mount directly to drive (i.e. /mnt/cache_ssd) not the user (/mnt/user)
  * See Unraid template settings - PHP memory and upload changes are important.
  * Postgressql and Redis are both needed. Redis caching enabled.
  * previews and cronjob will be setup.
  * Used Nextcloud-multimedia and nextcloud-cronjob docker containers.
### Unraid Template
![Nextcloud Unraid Template](https://github.com/ShyViolets/Server-Notes/blob/main/nextcloudtemplate.png?raw=true)

###Configuration File Edits
```
<?php
$CONFIG = array (
  'htaccess.RewriteBase' => '/',
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'apps_paths' => 
  array (
    0 => 
    array (
      'path' => '/var/www/html/apps',
      'url' => '/apps',
      'writable' => false,
    ),
    1 => 
    array (
      'path' => '/var/www/html/custom_apps',
      'url' => '/custom_apps',
      'writable' => true,
    ),
  ),
  'memcache.distributed' => '\\OC\\Memcache\\Redis',
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'redis' => 
  array (
    'host' => '192.168.1.112',
    'password' => '',
    'port' => 6379,
  ),
  'trusted_proxies' => 
  array (
    0 => '192.168.1.112',
  ),
  'overwrite.cli.url' => 'http://192.168.1.112:444',
  'overwriteprotocol' => 'https',
  'instanceid' => 'INSERTYOUROWN',
  'passwordsalt' => 'INSERTYOUROWN',
  'secret' => 'INSERTYOUROWN',
  'trusted_domains' => 
  array (
    0 => 'https://192.168.1.112:444',
    1 => 'YOUR.DOMAIN.COM',
  ),
  'datadirectory' => '/var/www/html/data',
  'dbtype' => 'pgsql',
  'version' => '25.0.5.1',
  'default_phone_region' => 'US',
  'dbname' => 'nextcloud',
  'dbhost' => '192.168.1.112:5432',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'dbuser' => 'USERNAME',
  'dbpassword' => 'PASSWORD',
  'installed' => true,
  'enable_previews' => true,
  'enabledPreviewProviders' => 
  array (
    0 => 'OC\\Preview\\TXT',
    1 => 'OC\\Preview\\MarkDown',
    2 => 'OC\\Preview\\OpenDocument',
    3 => 'OC\\Preview\\PDF',
    4 => 'OC\\Preview\\MSOffice2003',
    5 => 'OC\\Preview\\MSOfficeDoc',
    6 => 'OC\\Preview\\Image',
    7 => 'OC\\Preview\\Photoshop',
    8 => 'OC\\Preview\\TIFF',
    9 => 'OC\\Preview\\SVG',
    10 => 'OC\\Preview\\Font',
    11 => 'OC\\Preview\\MP3',
    12 => 'OC\\Preview\\Movie',
    13 => 'OC\\Preview\\MKV',
    14 => 'OC\\Preview\\MP4',
    15 => 'OC\\Preview\\AVI',
  ),
  'preview_max_memory' => 1024,
  'preview_max_x' => NULL,
  'preview_max_y' => NULL,
  'preview_max_filesize_image' => 400,
  'loglevel' => 2,
  'maintenance' => false,
  'app_install_overwrite' => 
  array (
    0 => 'discoursesso',
    1 => 'admin_notifications',
    2 => 'apporder',
  ),
  'memories.exiftool' => '/var/www/html/custom_apps/memories/exiftool-bin/exiftool-amd64-glibc',
  'twofactor_enforced' => 'false',
  'twofactor_enforced_groups' => 
  array (
  ),
  'twofactor_enforced_excluded_groups' => 
  array (
  ),
  'memories.vod.path' => '/var/www/html/custom_apps/memories/exiftool-bin/go-vod-amd64',
  'memories.vod.ffmpeg' => '/usr/bin/ffmpeg',
  'memories.vod.ffprobe' => '/usr/bin/ffprobe',
  'memories.gis_type' => 2,
  'memories.index.path' => '//Photos',
  'mail_from_address' => 'INSERTYOUROWN',
  'mail_smtpmode' => 'smtp',
  'mail_sendmailmode' => 'smtp',
  'mail_domain' => 'MAILDOMAIN',
  'mail_smtpauth' => 1,
  'mail_smtphost' => '192.168.1.112',
  'mail_smtpport' => '1025',
  'mail_smtpname' => 'EMAIL',
  'mail_smtppassword' => 'PASSWORD',
  
  'allow_local_remote_servers' => true,
    'app.mail.imap.timeout' => 20,
    'app.mail.smtp.timeout' => 20,
    'app.mail.sieve.timeout' => 20,
   ```
   ###    Administrative Settings
* All security checks should be passed. Troubleshoot anything you still need to do if it is not. Background job info is below.

![](https://github.com/ShyViolets/Server-Notes/blob/main/nextcloudsecuritypass.png?raw=true)

* Under basic settings, make sure background jobs is properly setup.

![](https://github.com/ShyViolets/Server-Notes/blob/main/administrationbasicsettings.png?raw=true)

   ### Nextcloud Cronjob Configuration
Will present with an error if ownership and permissions are not correct.  (if you messed up ownership settings somehow like I did during my setup). If not, skip. Otherwise, Follow below steps

1.  Make sure that the user is set to 99: in the template.
![](https://github.com/ShyViolets/Server-Notes/blob/main/cron1template.png?raw=true)
2. For both /mnt/user/nextcloud (data directory) and mnt/user/appdata/nextcloud (appdata directory) run the following command
```
 chown -R 99:100 ((insert directory here))
```
### Setting Up Preview Generator
[Youtube Tutorial](https://www.youtube.com/watch?v=qIhUE9IM_Uc)

In the console for Nextcloud-multimedia, run the following command.
![](https://github.com/ShyViolets/Server-Notes/blob/main/previews.png?raw=true)
```
php ./occ preview:generate-all -vvv 
```
Then add the following to cronjobs.

Name:

```occ-preview-pre-generate.sh```

Contents:

```php occ preview:pre-generate```
