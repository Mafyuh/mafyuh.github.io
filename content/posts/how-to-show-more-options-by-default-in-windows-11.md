---
date: 2023-09-05T23:45:25Z
description: ""
draft: false
slug: "how-to-show-more-options-by-default-in-windows-11"
tags:
  - "Windows"
title: "How to Show More Options By Default in Windows 11"
---

To 'Show more options' by default in File Explorer, open Command Prompt as Administrator, then type or paste the following command:

```
reg add HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32 /ve /d "" /f 
```

and hit Enter.

