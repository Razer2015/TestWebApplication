# Building Nginx which supports unsecure ciphers

## Building the image

```
docker build -t xfilefin/nginx-unsecure:dev .
```

Example config
```
server {
  listen 3000 ssl;

  ssl_certificate /etc/ssl/certs/certificate.crt;
  ssl_certificate_key /etc/ssl/certs/certificate.key;
  ssl_ciphers ALL:@SECLEVEL=0;
  ssl_protocols SSLv3 TLSv1.1 TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers on;

  location / {
    proxy_set_header Host $http_host;
    proxy_pass http://host.docker.internal:5244;
  }

  include /etc/nginx/extra-conf.d/*.conf;
}
```

_Well, that's pretty much it..._
