+++
date = 2023-10-29T16:20:00Z
description = ""
draft = false
slug = "how-to-authenticate-zammad-via-saml-with-nginx-proxy-manager"
title = "How to authenticate Zammad via SAML with Nginx Proxy Manager"
tags = ["authentik"]
showToc = false
+++


If you are getting error messages like:

```
422: the change you wanted was rejected. message from saml: actioncontroller::invalidauthenticitytoken
```

Just make sure you set these in your Nginx Proxy Manager hosts Advanced field:

```nginx
location / {
  proxy_pass http://zammad:8080; # Replace
  proxy_set_header  Host $host;
  proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header  X-Forwarded-Proto $scheme;
  proxy_set_header  X-Forwarded-Ssl on;
  proxy_set_header  X-Forwarded-Port $server_port;
  proxy_set_header  X-Forwarded-Host $host;
}
```

I spent way too long trying to figure this out, reading through Github issues, breaking my SAML provider and Zammad configs, starting over, when the whole time it was just good old nginx header issues.

Hope this helps someone out. Fix was found on this rails github issue.

(https://github.com/rails/rails/issues/22965)
