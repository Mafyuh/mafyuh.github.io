+++
date = 2023-09-30T16:20:00Z
description = ""
draft = false
slug = "how-to-authenticate-kasm-via-authentik"
title = "How To Authenticate KASM via authentik"
tags = ["authentik"]
+++

You could do this with OpenID as well but this method is using SAML. This guide assumes you already have running instances of Kasm Workspaces and authentik.

The official authentik docs dont have a Kasm Integration listed at the time. So I thought I would help out anyone who is trying to integrate these services via SAML. authentik's SAML docs can be found [here](https://goauthentik.io/integrations/sources/saml/).

## Setting up Kasm

In the Kasm Workspaces admin, click Access Management - Authentication - SAML and create a new configuration. Make sure you enable and make default after testing. You will probably find yourself switching between tabs alot, its best to start creating them both at the same time as you need links from each.

![KASM SAML Config](/assets/img/kasm-saml.png)

- Display Name: authentik
- Logo URL: https://auth.example.com/static/dist/assets/icons/icon.svg (or custom logo)
- Host Name: authentik
- NameID Attribute: emailAddress
- Entity ID: authentik
- Single Sign On Service/SAML 2.0 Endpoint: https://auth.example.com/application/saml/kasm/sso/binding/redirect/
- X509 Certificate: Skip to authentik setup first, then come back here. In authentik admin, go to your newly created SAML provider, when you click the provider and are brought to its details, you should have the option to Download signing certificate. Download it and paste the files contents here.

![authentik Config](/assets/img/kasm-auth.png)

## Setting up authentik

In the authentik admin, under Applications, create a new SAML provider. Once you created a provider, create an Application and set its provider to the newly created kasm provider. For simplicity sake, the provider and application name is kasm. (kasms pictured)

![authentik Config](/assets/img/kasm-auth2.png)

- Authorization flow: implicit
- ACS URL: https://kasm.example.com/api/acs/?id=e977b6cf72c7424328275db5f48fd15ab (Single Sign-On Service from kasm photo)
- Issuer: authentik (must be the same as Entity ID chosen in Kasm)
- Service Binding Provider: Post
- Audience: https://kasm.example.com/api/metadata/?id=e977b6cf72c7424328275db5f48fd15ab ( Entity ID URL from Kasm photo)
- Under Advanced, choose a signing certificate, default is authentik.
- Go back to Kasm x509 Certificate.

Make sure you save you changes. You should now be able to test SAML at the bottom of the page, once tested, I recommend opening a incognito tab and loading your KASM website.

![authentik Config](/assets/img/kasm-auth3.png)

You should now be able to authenticate yourself using SAML via authentik. I recommend going back to your admin session and adding your newly created user to the admin group. Also if it should only be you accessing this via authentik, I would change the kasm Application in authentik and bind it to your user.

Thank you to u/[agent-squirrel](https://www.reddit.com/user/agent-squirrel/) and this [subreddit](https://www.reddit.com/r/selfhosted/comments/vc30l7/kasm_authentik/) post on helping me with the NameID Attribute part!