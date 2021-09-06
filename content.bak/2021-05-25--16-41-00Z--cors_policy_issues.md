---
title: CORS Policy Issues
date: 2021-05-25 16:41
---

CORS
_Cross-Origin Resource Sharing_


## Resources

https://medium.com/@dtkatz/3-ways-to-fix-the-cors-error-and-how-access-control-allow-origin-works-d97d55946d9

CORS Wikipedia: https://en.wikipedia.org/wiki/Cross-origin_resource_sharing
Same-origin Policy Wikipedia: https://en.wikipedia.org/wiki/Same-origin_policy

CORS MDN Web Docs: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
Access-Control-Allow-Origin MDN Web Docs:
https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin

FreeCodeCamp "The Access-Control-Allow-Origin Header Explained – With a CORS
Example"
https://www.freecodecamp.org/news/access-control-allow-origin-header-explained/

Fetch standard: https://fetch.spec.whatwg.org

## Nginx Conf

### Wide Open CORS Conf
Create the file `/etc/nginx/conf.d/cors.conf` and add this to it. 

```
#
# Wide-open CORS config for nginx
#
location / {
     if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        #
        # Custom headers and headers various browsers *should* be OK with but aren't
        #
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 204;
     }
     if ($request_method = 'POST') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
     }
     if ($request_method = 'GET') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
     }
}
```

Source: https://enable-cors.org/server_nginx.html
Source: https://michielkalkman.com/snippets/nginx-cors-open-configuration/

## Nginx updates will overwrite /etc/nginx/vhosts
Here's how to edit the templates

https://wiki.inmotionhosting.com/index.php?title=Advanced_Ngxconf_Usage#Using_a_Custom_Template

Add this line to the appropriate part of the
/opt/ngxconf/templates.local/user_server.j2 template:

```
add_header Access-Control-Allow-Origin *;
```
