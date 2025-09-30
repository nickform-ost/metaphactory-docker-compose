# Readme Nginx

(for installation see [main readme](../README.md))

Run the Nginx compose setup from the instantiated `service-template`, e.g.

```
cp -r nginx/service-template nginx/config
cd nginx/config
docker-compose up -d
```


## Security hardening options

The templating definition allows to define security headers through environment variables in the container definition for `nginx-proxy` (see `docker-compose.overwrite.yml`). Alternatively, the environment variables can be specified on the virtual host, i.e. the metaphactory instance.

```
services:
  metaphactory:
    environment:
      - "SSL_POLICY=Mozilla-Modern"
```

The following environment variables are available:

Note: the value can be set to `off` to disable (i.e. not define the header). 

| Name                                | Description                                | Default                                                                                                                                                             |
|-------------------------------------|--------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| HSTS                                | Strict Transport Security                  | max-age=31536000; includeSubDomains; preload                                                                                                                        |
| X_FRAME_OPTIONS                     | X-Frame-Options header                     | DENY                                                                                                                                                                |
| CONTENT_SECURITY_POLICY             | Content-Security-Policy header             | default-src 'self'; script-src 'self' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; object-src 'none'; img-src 'self' https: data: blob:; font-src 'self' data:; |
| CONTENT_SECURITY_POLICY_REPORT_ONLY | Content-Security-Policy-Report-Only header | off                                                                                                                                                                 |
| X_CONTENT_TYPE_OPTIONS              | X-Content-Type-Option header               | nosniff                                                                                                                                                             |
| X_XSS_Protection                    | X-XSS-Protection header                    | off                                                                                                                                                                 |
| REFERRER_POLICY                     | Referrer-Policy header                     | same-origin                                                                                                                                                         |
| PERMISSIONS_POLICY                  | Permissions-Policy header                  | off                                                                                                                                                                 |


## Fallback redirect for unknown hosts


When an Nginx server is accessed via its IP address or a hostname it does not recognize (an unknown vhost), it is a security best practice to manage this traffic properly. Rather than displaying a generic 503 error, a better approach is to configure a fallback redirect to a known, valid domain. This prevents users from seeing a potentially untrusted self-signed certificate and steers them toward the correct system.

To achieve this, you can create a file named `_fallback.conf` inside the `conf.d` directory. This file contains a catch-all server block that listens for all requests not matched by other configurations. This ensures that any request to an unknown host gets processed by this block.

Here's an example of the configuration you would use:

```
server {
    server_name __;
    listen 443 ssl;
    listen 80;
    http2 on;
    ssl_certificate /etc/nginx/certs/myhost.example.com.crt;
    ssl_certificate_key /etc/nginx/certs/myhost.example.com.key;
    ssl_dhparam /etc/nginx/certs/myhost.example.com.dhparam.pem;
    return 301 https://myhost.example.com;
}
```

Notes:

* The certificate used in the fallback configuration should include the IP address in its respective fields (for browsers to accept it).
* The filename `_fallback.conf` is a convention to ensure it is loaded before other configurations. Nginx processes files in alphabetical order, so a filename starting with `_` or a similar character will be read first.


## Activating changed settings

* if environment variables in the container (see above) have been changed run `docker-compose up -d` to re-create the respective containers
* to re-generate and activate changed configuration, run `docker restart nginx-proxy`