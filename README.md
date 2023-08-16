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
