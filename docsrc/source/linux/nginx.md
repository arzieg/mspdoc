# nginx

## Links
https://devjournal.info/a-beginners-guide-to-nginx-proxy-manager/

## Reverse Proxy zum SUMA im OnPrem-Labor

Der Kommunikationsweg ist wie folgt: 
Azure Virtual Desktop -> NGINX -> SUMA Labor

!Das ist noch keine valide SSL Konfiguration, sondern eher der Ausgangspunkt, um überhaupt auf die WebUI zu gelangen!

1. Auf dem nginx-System die /etc/hosts pflegen: 

`<ip>  <suma-host-fqdn> <suma-host>`

2. Die default.conf pflegen (auf einem SUSE System liegt die unter /etc/nginx/vhost.d, unter Ubuntu in /etc/nginx/sites-available mit symlink zu /etc/nginx/sites-enabled). Hiermit werden alle http Anfragen an den anzusprechenden Host weitergeleitet. 

```
server {
    listen 80;
    #listen [::]:80 default_server ipv6only=on;
    return 301 https://$host$request_uri;
}
```

3. Server-Konfigurationsdatei

Erstellen einer Datei <hostname-fqdn>.conf - Datei für die Proxykonfiguration 

``` 
server {
        listen 443 ssl;  # Port, auf dem nginx lauscht
        server_name <suma-fqdn>;
        ssl_certificate           /etc/ssl/certs/<sumahost>.pem;
        ssl_certificate_key       /etc/ssl/private/<sumahost>.key;
 
        location / {
            proxy_pass https://<sumahost>/;  # Die URL des Backend-Servers
            proxy_set_header Host $host:$server_port;
            proxy_set_header REMOTE-HOST $remote_addr;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Forwarded-Server $host;
            proxy_set_header Accept-Encoding "";
            #proxy_set_header Referer $http_referer;
            #proxy_pass_request_body on;
            proxy_ssl_verify off;
            #proxy_ssl_server_name on;
        }
}

```

3. Die benötigten Zertifikatsdateien auf dem nginx-Server zur Verfügung stellen. Diese werden vom SUMA kopiert. 

4. nginx Konfiguration laden: `nginx -t; systemctl reload nginx`


## Quellen

https://www.digitalocean.com/community/tools/nginx   - Konfigurationstool für nginx