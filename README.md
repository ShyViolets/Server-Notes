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
