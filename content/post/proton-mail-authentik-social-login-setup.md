+++
date = 2023-11-12T16:20:00Z
description = ""
draft = false
slug = "proton-mail-authentik-social-login-setup"
title = "Proton Mail - SimpleLogin authentik Social Login Setup"

+++

This is just a quick guide on how to authenticate your authentik users with Proton using SimpleLogin OIDC.

![Proton Authentik Login Screen](/assets/img/proton-authentik.png)

To accomplish this, first create a [SimpleLogin](https://simplelogin.io/) acct by logging in with Proton. Once thats done go to https://app.simplelogin.io/developer and create a website. Give it your authentik URL.

Then go to Oauth Settings and copy your client ID and secret for next step. add your authentik URL in redirect URL like this https://auth.example.com/source/oauth/callback/simplelogin/ (simplelogin being slug of authentik)

In authentik go to Directory - Federation and Social login - Create and create an OpenID Oauth source

Name: SimpleLogin
Slug: simplelogin
User matching mode: i chose link with identical email
Consumer key: Paste your key
Consumer secret: Paste your secret
authorization url: https://app.simplelogin.io/oauth2/authorize
access token url: https://app.simplelogin.io/oauth2/token
profile url: https://app.simplelogin.io/oauth2/userinfo
OIDC Well-known URL: https://app.simplelogin.io/.well-known/openid-configuration


For logo, it appears authenik inverts your image, I dont know if its dark mode or bug but regardless here's the regular and inverted image I used. Just right click and save image:

![Proton Logo](/assets/img/proton-logo.png)
![Proton Logo Inverted](/assets/img/proton-inverted.png)

Now go to Flows and Stages - Flows - choose your default authentication stage - click it then click stage bindings - Click edit stage to the right of your identification stage - expand Source settings and make sure you CTL + click your newly created SimpleLogin source.

You should be able to logout and try to to login with your Proton account!