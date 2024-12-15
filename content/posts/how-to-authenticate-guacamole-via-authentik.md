---
date: 2023-10-29T16:20:00Z
description: ""
draft: false
slug: "how-to-authenticate-guacamole-authentik-nginxproxymanager"
title: "How to authenticate Guacamole via authentik with Cloudflare and Nginx Proxy Manager"
tags: 
  - "authentik"
showtoc: false
---

authentik's docs have a guide already for Guacamole. You can find that [here](https://goauthentik.io/integrations/services/apache-guacamole/). Follow all the instructions there, (especially the part where you create a user in Guacamole with the USERNAME of your email. not just filling in the email), but if you are using Cloudflare as our DNS you may run into problems. Such as infinite redirect loop.

## Error 403 Forbidden

While it was looping, I checked my Guacamole docker container logs in Portainer, and found the 403 Forbidden error.

```bash
22:03:59.418 [http-nio-8080-exec-2] INFO o.a.g.a.o.t.TokenValidationService - Rejected invalid OpenID token: JWT processing failed. Additional details: [[17] Unable to process JOSE object (cause: org.jose4j.lang.UnresolvableKeyException: Unable to find a suitable verification key for JWS w/ header {"alg":"RS256","kid":"xxx","typ":"JWT"} due to an unexpected exception (java.io.IOException: Non 200 status code (403 Forbidden) returned from https://example.com/application/o/guacamole/jwks/?exclude_x5) while obtaining or using keys from JWKS endpoint at https://example.com/application/o/guacamole/jwks/?exclude_x5): JsonWebSignature{"alg":"RS256","kid":"xxx","typ":"JWT"}
```

I assumed it had something to do with my Nginx Proxy Manager and the way I was proxying Guacamole, I do have WebSocket support and block common exploits enabled, their [docs](https://guacamole.apache.org/doc/gug/reverse-proxy.html) give a nginx config that I added under advanced.

```nginx
location /guacamole/ {
    proxy_pass http://HOSTNAME:8080;
    proxy_buffering off;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    access_log off;
}
```
I messed around with settings individually for hours, reading their docs, tried [oznu's](https://hub.docker.com/r/oznu/guacamole/) Guacamole image as well, this time with errors about the postgres version being incompatible. Figured it could be something with Cloudflare so turned down my HTTPS settings. Nada. Tried SAML, more errors.  Finally found this [github issue](https://github.com/goauthentik/authentik/issues/4082) and thanks to [Fma965](https://github.com/Fma965) for finding the solution.

Go to your Cloudflare Dashboard. Click on your domains summary and then on the left tab find **Rules**.

Under **Page Rules - Create a New Page Rule**, set the URL as your jwks URL from authentik's provider summary. Under pick a setting, choose Browser Integrity Check and make sure its unchecked. Save.

![Page Rules Images](/assets/img/pagerules.png)

This finally got me authenticated into my Guacamole instance via authentik. I spent way too much time on this integration. Anyways, hope this guide helps someone who may be in my shoes.